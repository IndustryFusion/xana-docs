# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

XANA is a field-technician support app: connect CRM and knowledge sources, open a **workbench** for a case, run an AI **investigation** (LangGraph + LLM), and guide the technician through evidence-backed repair steps with citations from CRM notes and PDF/DOCX manuals.

Four services, run independently (no root package.json / no single build):

| Service | Path | Port | Stack |
|---|---|---|---|
| Frontend | `frontend/` | 3000 | Next.js (App Router) |
| Backend | `backend/` | 4000 | NestJS, MongoDB |
| Workflow agent | `workflow-agent/` | 8000 | Python, FastAPI, LangGraph |
| Connector (web/storage) | `connectors/storages/web/` | 8080 | NestJS |
| CRM connector (Dynamics AX) | `connectors/CRM/dynamics-ax/` | 8081 | .NET, Docker |

Frontend talks only to the NestJS backend. The backend proxies to CRM/knowledge connectors, calls the workflow-agent for AI analysis, and calls a per-project configured LLM (via the `ai-providers` module — provider type/baseUrl/model/encrypted API key, not a hardcoded vendor) for connector field mapping. The workflow-agent calls back into the backend's REST API (`XanaApiClient`) for CRM/knowledge data rather than hitting connectors directly.

`docs/ARCHITECTURE.md` has a deep, mostly-accurate write-up (FigJam board, mermaid diagrams, honest per-block production-maturity scores) but predates the current MongoDB persistence and the current investigation-graph/hybrid-RAG design described below — treat its narrative framing as reliable, but verify specific file/module names against source before citing them.

## Running the stack

```bash
./dev.sh              # starts docker infra (Langfuse, Qdrant, Ollama, Dynamics connector) + all 4 local services in tmux session "xana-dev"
./dev.sh --no-docker   # skip docker infra (e.g. already running)
./dev.sh --no-langfuse # skip only the Langfuse tracing stack
./dev.sh --stop        # stop the tmux session (docker keeps running)
./dev.sh --status      # show docker + tmux status
tmux attach -t xana-dev
```

Each service can also be run standalone:

```bash
# backend (needs MONGODB_URI in backend/.env, see backend/.env.example)
cd backend && npm run start:dev

# frontend
cd frontend && npm run dev

# workflow-agent (needs venv + .env, see workflow-agent/.env.example)
cd workflow-agent && source .venv/bin/activate && uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# storage connector
cd connectors/storages/web && npm run build && npm run start:prod
```

Qdrant (required for workflow-agent RAG) via `docker compose up qdrant -d` from `workflow-agent/`.

## Build, lint, test

**Backend** (`backend/`, NestJS + Jest):
```bash
npm run build          # nest build
npm run lint           # eslint --fix
npm test               # jest, all *.spec.ts under src/
npx jest src/proxy/proxy.service.spec.ts     # single file
npx jest -t "test name substring"            # single test by name
npm run test:e2e       # jest --config ./test/jest-e2e.json
npm run migrate:mongo  # one-time migration from legacy backend/.data/*.json into MongoDB
```

**Frontend** (`frontend/`, Next.js):
```bash
npm run build
npm run lint
```
No frontend test suite exists yet.

**Workflow agent** (`workflow-agent/`, pytest, `testpaths = tests`):
```bash
pytest                              # all tests
pytest tests/test_rag_pipeline.py   # single file
pytest tests/test_rag_pipeline.py::test_name -v   # single test
```

**Storage connector** (`connectors/storages/web/`, NestJS + Jest):
```bash
npm run build && npm run lint && npm run typecheck && npm test
```

## Backend architecture (`backend/src/`)

NestJS modules, each roughly one concern:

- `auth` — DB-backed users (MongoDB `users` collection) with hashed passwords and role-gated CRUD; sessions use a custom HMAC-signed token (not a real JWT). `APP_ADMIN_USERNAME`/`PASSWORD` only seed the first admin account on startup, they aren't the ongoing auth source
- `connectors` — register CRM/knowledge connectors via the OpenXANA manifest contract (`GET {baseUrl}/openxana/manifest`), AI-assisted field mapping
- `proxy` — HTTP proxy to connector base URLs, with a response cache (`connector_response_cache` collection, TTL-based)
- `knowledge` — browse connector folders, download PDF/DOCX content
- `support` — cases, accounts, per-case product mappings (`case_support_state`)
- `workbenches` — workbench CRUD; bridges to workflow-agent for analyze/continue
- `projects` — `project.selectedSkillId` picks which LangGraph skill graph a workbench uses (via workflow-agent's `registry.py`)
- `skills` — catalog of available skills
- `ontology` — domain-ontology graphs (classes/edges), unrelated to skill selection; consumed inside the graph's `load_ontology` node
- `ai-providers` — pluggable per-project LLM provider config (encrypted API key)
- `mcp-servers` — generic OAuth2/PKCE client for whatever MCP server you register; no provider-specific code
- `database` — Mongo connection/schemas; `json-migration.service.ts` handles the legacy JSON→Mongo migration path

Persistence is **MongoDB** (`MONGODB_URI`), not JSON files. `backend/src/database/schemas/` has 11 schemas — including `workbenches`, `connectors`, `connector_response_cache`, `case_support_state`, `connector_mappings`, `users`, `projects`, `skills`, `ai_providers`, `mcp_servers`, `global_ontologies` — check that directory rather than assuming `backend/README.md`'s table is exhaustive. (Older docs/commit history reference `backend/.data/*.json`; that's the pre-migration state, kept only as a one-time import source.)

## Workflow-agent architecture (`workflow-agent/src/`)

LangGraph service, invoked by the backend's `WorkbenchesService` via `XanaApiClient` (`POST /v1/workbench/analyze`, `POST /v1/workbench/continue`). It does not talk to connectors directly — it re-fetches CRM/knowledge data from the NestJS backend, which already resolved manifest paths.

Graphs are **skill-scoped**, not monolithic: `src/graphs/registry.py` maps a `skill_id` to a (analysis, continue, synthesis) graph triple. Currently one skill is implemented: `src/graphs/skills/metal_processing_support/` (analysis_graph.py, continue_graph.py, synthesis_graph.py, prompts.py). New skills register into the same registry.

The analysis graph (`metal_processing_support/analysis_graph.py`) is an iterative investigation loop, not a straight pipeline:
```
load_ontology → enrich_context → retrieve_passages → supplement_web → build_evidence_map
  → generate_observations_and_hypotheses → evaluate_confidence
  → (retrieve_more_knowledge → back to generate_observations_and_hypotheses) | select_next_item | finalize_recommendation
```
Deliberately no resolve/ingest step here — `analyze()` only queries what a project's knowledge scope has already had indexed into Qdrant/BM25 (via the reindex flow, `POST /v1/knowledge/index-project`); live-resolving and re-ingesting on every single analyze/continue call was a major source of latency and connector-driven timeouts. `generate_observations_and_hypotheses` is itself an agentic tool-calling step (seven fixed tools, `graphs/tools/chat_tools.py`/`graphs/llm.py`), not a single LLM call over a pre-fetched packet — see `docs/ARCHITECTURE.md` §1-2 for the full, diagrammed breakdown.

RAG (`src/rag/`) is **hybrid**, not keyword-only: Qdrant vector search + BM25 (`sparse_index.py`) fused via RRF, reranked, MMR for diversity (`vector_store.py`, `retriever.py`, `rerank.py`). The BM25 index is built once per retrieval call (not once per query variant) and cached in-process across calls within the same investigation scope (`sparse_index.get_or_build_bm25_index`); candidate/rerank pool sizes scale with scoped corpus size rather than being flat constants (`retriever.derive_candidate_pool`, `rerank._derive_rerank_pool`, bounds in `config.py`). Document ingest (`document_ingest.py`, `pdf_ingest.py`) handles PDF/DOCX/TXT, with Tesseract OCR for image-heavy PDF pages and optional Ollama vision captioning for diagrams (ingest-time only). Technician-uploaded photos/video use a separate OCR path (LightOnOCR via the same Ionos API) in `step_media_ocr.py`/`media_processor.py`. Optional Tavily web search (`web_search.py`) supplements thin manual coverage — gated, never replaces scoped manuals.

`src/observability/` wires Langfuse tracing (`workflow-agent/langfuse/docker-compose.yml`, UI on :3001).

## Frontend structure (`frontend/`)

Next.js App Router. `app/[workspace]/...` is the workspace-scoped area (workbenches, fusionpass, process-twin); `app/admin` and `app/developer` are separate top-level areas. `lib/api.ts` is the single client for all backend calls — check it first when tracing a frontend→backend call. Per `docs/ARCHITECTURE.md`, some surfaces are still mock/demo-only (e.g. FusionPass/Process Twin incident lists) — don't assume every UI affordance is wired to a real backend call without checking `lib/api.ts` and the relevant page. (The `app/[workspace]/accounts/` case-workspace area and its backing `backend/src/resolution/` module — an older, separate, CRM-case-keyed AI analysis system, unrelated to the workbench/LangGraph investigation flow — were removed: the route had no navigation entry point anywhere in the app and was confirmed unreachable/orphaned before deletion.)

## Connectors (`connectors/`)

External systems, each a separate deployable, registered with the backend via OpenXANA manifest URL:
- `connectors/CRM/dynamics-ax/` — .NET adapter in front of a Dynamics-style CRM, has its own Dockerfile/k8s manifests
- `connectors/storages/web/` — NestJS document/wiki storage connector
- `connectors/ERP/` — currently empty (`.gitkeep` only)

## Cross-cutting notes

- `WORKFLOW_AGENT_ENABLED=false` (or workflow-agent unreachable/timed out) makes the backend save a placeholder `AnalysisRun` (`isFallback: true`, no citations, no steps, `reasoningBrief` says analysis couldn't complete) rather than generating rule-based steps — the technician has to reanalyze once the agent is available.
- Frontend EN/DE language selection is passed through as `language` on analyze/continue calls, overriding the workflow-agent's default.
- `backend/.env` and `workflow-agent/.env` share the same `OPENAI_COMPATIBLE_*` LLM vars; see each `.env.example` for the full list before assuming a var name.
- Auth has moved past the single-env-admin design `docs/ARCHITECTURE.md` describes (see finding above), but only 9 of 18 backend controllers reference `AuthService` at all — don't assume a given route is guarded without checking its controller. There is no global auth guard (`app.module.ts` only registers a request-logging middleware for all routes); see `docs/architecture/backend.md`'s "Auth coverage" section for the current list of which controllers are and aren't guarded.
- **Sales/support isolation**: `sales` and `support` are separate workspace verticals with separate purposes and must stay isolated at the code level. Do not modify any file under `backend/src/support/`, `backend/src/workbenches/`, `frontend/app/support/`, or workflow-agent's `metal_processing_support` skill as part of sales work, and vice versa, without the user's explicit consent. Elements are reusable across verticals only when the user explicitly names them as reusable (e.g. the Dynamics AX CRM connector, `ProxyService`/`ConnectorsService`, generic Mongo/auth infra) — and even then, changes to shared/reused code must be strictly additive (new fields/resources/endpoints) and must not alter behavior that `support` currently depends on. This extends down to the connector level: `connectors/CRM/dynamics-ax/SalesService.cs`/`SalesController.cs` are wholly separate files from `DynamicsService.cs`, reusing only its ADFS auth pattern, not its code.
- `chunking_overlap_chars` and `chunking_dedup_enabled` (`workflow-agent/src/config.py`) are implemented and tested (adjacent-chunk overlap, within-document exact-duplicate dropping — `chunking.py: _apply_overlap`/`dedup_chunks`) but both default to their no-op value (`0`/`False`) — don't assume they're dead code just because they're inert by default. Turning either on for real requires bumping `CHUNKING_SCHEMA_VERSION` (`chunking.py`) in the same change, since that's what invalidates already-cached content; the per-project "Reingest knowledge base → Restart from scratch" button (`frontend/app/support/page.tsx`, `POST /projects/:id/reingest`) is the intended way to then re-chunk a specific project's knowledge scope on demand, not a global/automatic rollout.
