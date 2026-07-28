# Workflow agent

> [docs](../README.md) → [Architecture](../ARCHITECTURE.md) → Workflow agent
> See also: [`workflow-agent/README.md`](../../workflow-agent/README.md) and [`workflow-agent/INTEGRATION.md`](../../workflow-agent/INTEGRATION.md) for local run and backend-wiring detail.

Python, FastAPI, LangGraph, port 8000. This is where the actual AI investigation happens. It does not talk to connectors directly — it re-fetches CRM/knowledge data from the NestJS backend, which has already resolved manifest paths and field mappings.

## Skill-scoped graphs

Graphs are scoped per **skill**, not one monolithic pipeline: `src/graphs/registry.py` maps a `skill_id` to an (analysis, continue, synthesis) graph triple. A [project](../guides/05-projects.md) picks which skill its workbenches use. Today's shipped skill is metal-processing support (`src/graphs/skills/metal_processing_support/`); new skills register into the same registry.

**The full technical deep-dive on this service now lives in the main architecture document, not here** — it's the core the whole app is built around, so it gets the detailed treatment there:

- [§1 — The core: the workflow agent](../ARCHITECTURE.md#1-the-core-the-workflow-agent) — the analysis graph's node flow, and how it and the continue graph share one agentic reasoning core
- [§2 — Tool calling, connectors, and MCP](../ARCHITECTURE.md#2-connecting-to-data-sources-tool-calling-connectors-and-mcp) — the seven tools the reasoning LLM can call, and MCP's actual (not-yet-wired-in) status
- [§3 — Configuring the LLM](../ARCHITECTURE.md#3-configuring-the-llm) · [§4 — Embeddings and the vector store](../ARCHITECTURE.md#4-embeddings-and-the-vector-store) · [§5 — OCR and document parsing](../ARCHITECTURE.md#5-ocr-and-document-parsing) · [§6 — The RAG pipeline](../ARCHITECTURE.md#6-the-rag-pipeline--tying-it-together)

## API surface (`src/main.py`)

| Endpoint | Purpose |
|---|---|
| `GET /health` | Service + RAG status |
| `POST /v1/workbench/analyze` | Initial repair-step workflow — §1's analysis graph |
| `POST /v1/workbench/continue` | Chat / step-feedback continuation — §1's continue graph |
| `POST /v1/workbench/synthesize-resolution` | Generate the resolution summary/root-cause text when a workbench is resolved |
| `POST /v1/knowledge/index` | Background-index a set of knowledge leaves |
| `POST /v1/knowledge/index-project` | Index (or reindex) an entire project's knowledge scope — what the frontend's per-project **Reingest** control ([5. Projects](../guides/05-projects.md)) actually calls |
| `POST /v1/knowledge/ensure-collection` | Create a workspace's Qdrant collection if it doesn't exist yet |
| `GET /v1/knowledge/index-status/{workspaceId}` | Ingestion progress/status for a workspace (what the reingest UI polls) |
| `DELETE /v1/knowledge/index/{workspaceId}/{fileId}` | Remove one file's vectors |
| `DELETE /v1/knowledge/collection/{workspaceId}` | Drop an entire workspace's Qdrant collection |
| `GET /v1/knowledge/evidence-asset/{assetId}` | Fetch a stored evidence asset (e.g. an extracted page/diagram image) |
| `GET /v1/knowledge/page-image` | Fetch a rendered manual page image, for the in-chat page viewer |
| `GET` / `PUT /v1/config/llm` | Read/override the LLM config the agent is using — a runtime, non-per-request config path distinct from the per-request `AiProviderConfig` in [ARCHITECTURE.md §3](../ARCHITECTURE.md#3-configuring-the-llm) |
| `POST /v1/media/process-image` / `process-url` / `process-video` | Run OCR (LightOnOCR, [§5](../ARCHITECTURE.md#5-ocr-and-document-parsing)) on a technician-uploaded photo, URL, or video — the trigger side of `read_attachment_details` ([§2](../ARCHITECTURE.md#2-connecting-to-data-sources-tool-calling-connectors-and-mcp)) |

## Observability

`src/observability/` wires Langfuse tracing (UI on :3001) so an investigation's tool calls, retrieval, and LLM steps are inspectable end to end.

## Where to go next

- Who calls it, and how the result is used: [Backend](backend.md), [Workbenches guide](../guides/06-workbenches.md)
