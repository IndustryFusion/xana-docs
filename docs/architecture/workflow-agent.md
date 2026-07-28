# The investigation engine

> [Documentation](/docs/README.md) → [Architecture](/docs/ARCHITECTURE.md) → The investigation engine
> See also: [service details](/workflow-agent/README.md) and [how the engine and the hub work together](/workflow-agent/INTEGRATION.md).

**Python, FastAPI, LangGraph**, listening on port 8000. This is where the actual AI investigation happens. It does not talk to connectors directly — it re-fetches CRM/knowledge data from the hub, which has already resolved manifest paths and field mappings.

## Skill-scoped graphs

Graphs are scoped per **skill**, not one monolithic pipeline: a registry maps a skill id to an (analysis, continue, synthesis) graph triple. A [project](/docs/guides/05-projects.md) picks which skill its workbenches use. Today's shipped skill supports industrial equipment maintenance and repair; new skills register into the same registry without disturbing the ones already running.

**The full technical deep-dive on this service lives in the main architecture document, not here** — it's the core the whole app is built around, so it gets the detailed treatment there:

- [§1 — The core: the investigation engine](/docs/ARCHITECTURE.md#_1-the-core-the-investigation-engine) — the analysis graph's node flow, and how it and the continue graph share one agentic reasoning core
- [§2 — Tool calling, connectors, and MCP](/docs/ARCHITECTURE.md#_2-connecting-to-your-data-tools-and-connectors) — the seven tools the reasoning LLM can call, and MCP's actual (not-yet-wired-in) status
- [§3 — Configuring the LLM](/docs/ARCHITECTURE.md#_3-bringing-your-own-ai-model) · [§4 — Embeddings and the vector store](/docs/ARCHITECTURE.md#_4-how-your-knowledge-base-becomes-searchable) · [§5 — OCR and document parsing](/docs/ARCHITECTURE.md#_5-reading-manuals-photos-and-diagrams) · [§6 — The RAG pipeline](/docs/ARCHITECTURE.md#_6-the-retrieval-pipeline-tying-it-together)

## API surface

| Endpoint | Purpose |
|---|---|
| `GET /health` | Service + retrieval-pipeline status |
| `POST /v1/workbench/analyze` | Initial repair-step workflow — §1's analysis graph |
| `POST /v1/workbench/continue` | Chat / step-feedback continuation — §1's continue graph |
| `POST /v1/workbench/synthesize-resolution` | Generates the resolution summary/root-cause text when a workbench is resolved |
| `POST /v1/knowledge/index` | Background-indexes a set of knowledge leaves |
| `POST /v1/knowledge/index-project` | Indexes (or reindexes) an entire project's knowledge scope — what the interface's per-project **Reingest** control calls |
| `POST /v1/knowledge/ensure-collection` | Creates a project's Qdrant collection if it doesn't exist yet |
| `GET /v1/knowledge/index-status/{workspaceId}` | Ingestion progress/status for a project (what the reingest UI polls) |
| `DELETE /v1/knowledge/index/{workspaceId}/{fileId}` | Removes one file's vectors |
| `DELETE /v1/knowledge/collection/{workspaceId}` | Drops an entire project's Qdrant collection |
| `GET /v1/knowledge/evidence-asset/{assetId}` | Fetches a stored evidence asset (e.g. an extracted page/diagram image) |
| `GET /v1/knowledge/page-image` | Fetches a rendered manual-page image, for the in-chat page viewer |
| `GET` / `PUT /v1/config/llm` | Reads/overrides the runtime LLM configuration the engine is using — distinct from the per-request AI-provider configuration in [ARCHITECTURE.md §3](/docs/ARCHITECTURE.md#_3-bringing-your-own-ai-model) |
| `POST /v1/media/process-image` / `process-url` / `process-video` | Runs OCR on a technician-uploaded photo, URL, or video — the trigger side of `read_attachment_details` ([§2](/docs/ARCHITECTURE.md#_2-connecting-to-your-data-tools-and-connectors)) |

## Observability

**Langfuse** tracing (UI on port 3001) instruments every investigation's tool calls, retrieval, and LLM turns end to end, so an unexpected answer is diagnosable rather than a black box.

## Where to go next

- Who calls it, and how the result is used: [The hub](/docs/architecture/backend.md), [Workbenches guide](/docs/guides/06-workbenches.md)
