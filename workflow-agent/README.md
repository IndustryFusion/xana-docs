# XANA Workflow Agent

LangGraph-powered troubleshooting workflows for XANA Business Support Workbenches.

## What it does

- Receives a workbench snapshot from the NestJS backend
- Uses **tools** to enrich context from XANA Business APIs:
  - CRM case context (`/support/.../cases/.../context`)
  - Knowledge folders/files (`/knowledge/...`)
  - Pipeline incident refs from the workbench snapshot
- Runs LangGraph workflows with **hybrid RAG** (Qdrant vectors + BM25 + rerank + MMR)
- Parses knowledge PDFs/DOCX/TXT into a file cache under `data/rag-cache/`
- Returns cited evidence and troubleshooting steps grounded in manual passages

### RAG stack

1. Resolve knowledge scope → download/parse → file cache
2. Index chunks into Qdrant (per workspace collection)
3. Retrieve with vector search (scoped by file IDs) + BM25 fused via RRF
4. Rerank top candidates and apply MMR for diversity
5. Inject passages into LLM context packets

Start Qdrant locally:

```bash
docker compose up qdrant -d
```

### Technician media OCR (LightOnOCR)

Uploaded step-help photos, workbench photo logs, and pipeline incident images use **LightOnOCR-2-1B** via the same Ionos OpenAI-compatible API as the chat LLM. PDF manual ingest still uses Tesseract.

```bash
# In .env (reuses OPENAI_COMPATIBLE_BASE_URL and OPENAI_COMPATIBLE_API_KEY)
OCR_MODEL=lightonai/LightOnOCR-2-1B
OCR_ENABLED=true
OCR_FALLBACK_TESSERACT=true
```

- **Upload path:** `POST /v1/media/process-image` and `process-video` run LightOnOCR (per frame for video), with optional Tesseract fallback.
- **Step chat:** before the copilot reply, step-scoped images are re-OCR'd with LightOnOCR if needed.
- **Reasoning:** the text LLM (`OPENAI_COMPATIBLE_MODEL`) reads `ocrText` markdown — it does not receive raw pixels.

LightOnOCR is document OCR, not general scene description. Non-text field photos may return little text; the copilot should ask the technician to describe what they see.

### Web search supplement (optional)

When manual RAG is thin, the agent can call **Tavily** to fetch external passages into `retrievedWebChunks`. Web never replaces scoped manuals.

```bash
WEB_SEARCH_ENABLED=true
TAVILY_API_KEY=tvly-...
WEB_SEARCH_TRIGGER_MIN_MANUAL_SCORE=0.35
```

Safeguards: gated trigger, citation verification against fetched chunk text, `trustTier official` for vendor/docs domains, forum sources downgraded to hints (procedural steps scrubbed).

### PDF OCR (optional but recommended)

Install Tesseract on the host for OCR on image-heavy **manual PDF pages** during RAG ingest:

```bash
sudo apt-get install -y tesseract-ocr tesseract-ocr-deu
```

The workflow agent OCRs embedded PDF images and standalone PNG/JPG in knowledge scope with Tesseract. Technician uploads use LightOnOCR when configured.

## Run locally

```bash
cd workflow-agent
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# Edit .env — set OPENAI_* and QDRANT_URL
uvicorn src.main:app --host 0.0.0.0 --port 8000 --reload
```

Health check (includes RAG status):

```bash
curl http://localhost:8000/health
```

## Run with Docker

```bash
cd workflow-agent
cp .env.example .env
docker compose up --build
```

When the NestJS backend runs on the host, use:

```
XANA_API_BASE_URL=http://host.docker.internal:4000
```

## API

| Endpoint | Purpose |
|----------|---------|
| `GET /health` | Service + RAG status |
| `POST /v1/workbench/analyze` | Initial repair-step workflow |
| `POST /v1/workbench/continue` | Help / failed-step continuation |
| `POST /v1/knowledge/index-workbench` | Pre-index all manuals from a workbench knowledge scope |
| `POST /v1/knowledge/index` | Background index knowledge leaves |
| `DELETE /v1/knowledge/index/{workspaceId}/{fileId}` | Remove vectors for a file |

See [INTEGRATION.md](./INTEGRATION.md) for backend/frontend wiring.
