# Docker deployment — XANA full stack

Run the entire XANA product from the repository root. All `docker compose` commands below are run from the repo root, not from `docs/`.

> Part of [docs/](../README.md) → [Architecture](../ARCHITECTURE.md). See also the [root README](../../README.md).

## Prerequisites

- Docker Engine with Compose v2
- Copy environment template: `cp .env.example .env` and set LLM + connector secrets

## Quick start

### Core stack (UI + API + MongoDB + workflow-agent + Qdrant)

```bash
docker compose up --build
```

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:4000 |
| Backend health | http://localhost:4000/health |
| Workflow agent | http://localhost:8000/health |
| MongoDB | localhost:27017 |
| Qdrant | http://localhost:6333 |

Default login: `admin` / `admin123` (override via `APP_ADMIN_*` in `.env`).

### Full stack with OpenXANA connectors

Requires CRM and wiki credentials in `.env`:

```bash
docker compose --profile connectors up --build
```

| Connector | Host URL | Internal URL (backend proxy) |
|-----------|----------|------------------------------|
| Web knowledge | http://localhost:8080 | http://web-connector:8080 |
| Dynamics CRM | http://localhost:8081 | http://dynamics-connector:8080 |

When `SEED_CONNECTORS=true` (default), a one-shot `seed-connectors` service registers both connectors for workspace `support-ai` in MongoDB.

## Profiles

| Profile | Services |
|---------|----------|
| *(default)* | mongo, qdrant, backend, frontend, workflow-agent |
| `connectors` | web-connector, dynamics-connector, seed-connectors |

## Environment

All variables are documented in [`.env.example`](../../.env.example) (repo root). Key values:

- `NEXT_PUBLIC_API_URL` — browser → backend (keep `http://localhost:4000` when using published ports)
- `OPENAI_COMPATIBLE_*` — LLM for workbench AI and connector mapping
- `CRM_*` — Dynamics connector credentials
- `WIKI_USERNAME` / `WIKI_PASSWORD` — substituted into mounted web connector config

## Volumes

| Volume | Purpose |
|--------|---------|
| `mongo_data` | MongoDB persistence |
| `qdrant_data` | Vector store |
| `backend_uploads` | Workbench attachment uploads |
| `workflow_agent_data` | LangGraph checkpoints + RAG cache |

## Agent-only development

For workflow-agent + Qdrant without the full stack, use the existing compose file:

```bash
cd workflow-agent
docker compose up --build
```

That stack expects the NestJS backend on the host at port 4000 (`host.docker.internal`).

## Project Dockerfiles

| Project | Dockerfile |
|---------|------------|
| Backend | `backend/Dockerfile` |
| Frontend | `frontend/Dockerfile` |
| Workflow agent | `workflow-agent/Dockerfile` |
| Web connector | `connectors/storages/web/Dockerfile` |
| Dynamics CRM | `connectors/CRM/dynamics-ax/Dockerfile` |

## Kubernetes / Rancher Fleet deployment

The GitOps chart set for this stack lives in a sibling `GitOpsRepo` checkout (outside this repository), kept as the dedicated deployment repository for the environment pipeline.

## Troubleshooting

- **Frontend cannot reach API** — confirm `NEXT_PUBLIC_API_URL=http://localhost:4000` and rebuild frontend (`docker compose build frontend`).
- **CORS errors** — set `CORS_ORIGINS=http://localhost:3000` in `.env`.
- **Connectors profile fails** — ensure `CRM_URL`, `CRM_USERNAME`, and `CRM_PASSWORD` are set.
- **AI analyze fails** — set `OPENAI_COMPATIBLE_*` in `.env` and restart `workflow-agent`.
- **Skip connector seeding** — set `SEED_CONNECTORS=false` in `.env`.
