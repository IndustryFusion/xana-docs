# 1. Setup

> [docs](../README.md) → [Guides](README.md) → Setup
> Next: [2. First login](02-first-login.md)

Two ways to run XANA locally: the native dev launcher (`dev.sh`, hot-reload, recommended day-to-day), or full Docker Compose (closer to production). Both need the same secrets in `.env` first.

## 1. Prerequisites

- Node.js + npm (frontend, backend)
- Python 3.12 + a virtualenv tool (workflow-agent)
- Docker Engine with Compose v2 (for MongoDB, Qdrant, Langfuse, and the Dynamics connector — and for the full-Docker option below)
- tmux (only needed by `dev.sh`)

## 2. Configure secrets

```bash
cp .env.example .env
```

Fill in, at minimum:

| Variable | Why |
|---|---|
| `APP_ENCRYPTION_KEY` | 32-byte hex value the backend uses to encrypt stored secrets (AI provider API keys, connector credentials). Generate with `openssl rand -hex 32`. Required — the backend will not start without it. |
| `APP_ADMIN_USERNAME` / `APP_ADMIN_PASSWORD` | Seeds the **first** admin account on first startup only (see [2. First login](02-first-login.md)). |
| `OPENAI_COMPATIBLE_BASE_URL` / `_API_KEY` / `_MODEL` | An OpenAI-compatible LLM endpoint. Needed for AI workbench analysis and connector field mapping. Per-project overrides come later via the `ai-providers` admin screen — this `.env` value is the fallback/default. |
| `MONGODB_URI` (in `backend/.env`, see `backend/.env.example`) | All durable backend state lives in MongoDB — connectors, workbenches, projects, users, ai-providers, mcp-servers, ontologies. |

The rest of `.env.example` (Qdrant, OCR, web search, retrieval tuning) has working defaults — adjust only if you know you need to.

## 3. Start supporting infrastructure + all services

```bash
./dev.sh
```

This starts Docker infra (Langfuse, Qdrant, Ollama, and the Dynamics connector) and then the four local services — frontend, backend, workflow-agent, storage connector — each in its own hot-reloading tmux window, in a session named `xana-dev`.

**`dev.sh` does not start MongoDB** — the backend needs one reachable at whatever `MONGODB_URI` in `backend/.env` points to before it will start. If you don't already have a MongoDB to point at, start the standalone one from the root compose file: `docker compose up mongo -d` (leaves it running independently of `dev.sh`).

```bash
tmux attach -t xana-dev      # reattach after detaching
# Ctrl+B then <window number>  jump to a specific service
# Ctrl+B then D                detach (leaves everything running)

./dev.sh --no-docker    # skip the docker infra step (e.g. already running)
./dev.sh --no-langfuse  # skip only the Langfuse tracing stack
./dev.sh --stop         # stop the tmux session (docker keeps running)
./dev.sh --status       # show docker + tmux status
```

Qdrant is required for the workflow-agent's hybrid RAG retrieval — `dev.sh` starts it for you; standalone it's `docker compose up qdrant -d` from `workflow-agent/`.

### Or: run everything in Docker

For a deployment-like local run instead of hot-reload dev:

```bash
docker compose up --build
```

See [Docker deployment](../setup/docker-deployment.md) for compose profiles (including the `connectors` profile for CRM/knowledge connectors), ports, volumes, and troubleshooting.

## 4. Verify it's up

| Service | Check |
|---|---|
| Frontend | http://localhost:3000 |
| Backend | http://localhost:4000/health |
| Workflow agent | http://localhost:8000/health |

If all three respond, continue to [2. First login](02-first-login.md).
