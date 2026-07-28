# XANA Business

XANA Business is an AI-agent platform installed inside a company to connect its existing data sources — CRM and knowledge bases (manuals, wikis, documents) today, with ERP planned (`connectors/ERP/` is a placeholder, not yet implemented) — and turn them into AI-powered **workspaces** for different parts of the business.

Today that's:

- **Service & Support** — the most complete workspace. Connect a CRM and knowledge sources, open a **workbench** for a case, run an AI **investigation** that produces an evidence-backed, step-by-step repair guide with citations back to CRM notes and manuals.
- **Sales** — an early-stage workspace, starting with an automated weekly sales report generated from CRM appointment data.
- **Operations** — on the roadmap, not yet built.
- **Academy** — a longer-term workspace idea, not yet built.

Inside a workspace, an admin creates one or more **projects** — each project scopes a connector, an AI provider, a knowledge base, and (in Service & Support) a skill graph. Inside a project, users create **workbenches**, one per task. In Service & Support, for example, a workbench is tied to a CRM case (and optionally a FusionPass / Process Digital Twin incident) and drives the AI investigation into a step-by-step troubleshooting guide.

## Contents

*(kept up to date as documentation is added — see [docs/README.md](docs/README.md) for the full index)*

- [Quick start](#quick-start)
- [Deployment to Kubernetes](#deployment-to-kubernetes)
- [1. Architecture](#1-architecture)
- [2. Frontend](#2-frontend)
- [3. Backend](#3-backend)
- [4. Workflow agent](#4-workflow-agent)
- [5. Connectors](#5-connectors)
- [What's still pending](#whats-still-pending)
- [Guides — using XANA](docs/guides/README.md) — setup → first login → users → connectors → projects → workbenches → sales module
- [Docker deployment](docs/setup/docker-deployment.md)

## Quick start

```bash
./dev.sh               # docker infra (Langfuse/Qdrant/Ollama/Dynamics connector — not MongoDB, see below) + all 4 local services, hot-reload, in tmux session "xana-dev"
./dev.sh --no-docker    # skip the docker infra step (e.g. already running)
./dev.sh --no-langfuse  # skip only the Langfuse tracing stack
./dev.sh --stop         # stop the tmux session (docker keeps running)
./dev.sh --status       # show docker + tmux status
tmux attach -t xana-dev # reattach after detaching
```

That's the native/hot-reload path. For a deployment-like run instead, `docker compose up --build` — see [Docker deployment](docs/setup/docker-deployment.md). Full prerequisites and `.env` setup: [1. Setup](docs/guides/01-setup.md).

## Deployment to Kubernetes

Beyond local dev, XANA runs on Kubernetes as one Helm chart per service, reconciled via Rancher Fleet (GitOps continuous delivery) rather than anyone running `kubectl`/`helm` by hand against the cluster.

The Kubernetes manifests themselves are **not part of this repository** — they live in a separate, dedicated deployment repo: **[GitOpsRepo](https://github.com/IndustryFusion/GitOpsRepo)** (a sibling checkout next to this one, `../GitOpsRepo`). Application code (here) and deployed cluster state (there) are versioned and reviewed independently, on purpose.

→ [docs/setup/kubernetes-deployment.md](docs/setup/kubernetes-deployment.md) for how GitOpsRepo is structured, deploy order, secrets, and a manual (no-Fleet) install.

---

Below is a point-by-point tour of the five things that make up this repo — what each one does and how it fits with the rest. Each point links to its full write-up in `docs/`.

## 1. Architecture

The technical core of this application: the LangGraph workflow agent, how its reasoning loop calls tools to reach CRM/knowledge data and MCP servers, how the LLM/embedding model/OCR are configured, and the hybrid RAG pipeline that grounds every answer in a citation.

→ [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

## 2. Frontend

The Next.js (App Router) UI — the only thing a user ever opens, and the only piece that talks to the backend directly. Renders the workspace selector, the Service & Support project/workbench pages, the Sales report config page, and the Admin area.

→ [docs/architecture/frontend.md](docs/architecture/frontend.md)

## 3. Backend

The NestJS API hub, backed by MongoDB. Everything durable — users, projects, connectors, workbenches, sales reports, AI-provider config — lives here. It's the only piece that talks to connectors and the workflow-agent.

→ [docs/architecture/backend.md](docs/architecture/backend.md)

## 4. Workflow agent

The Python/FastAPI/LangGraph service that actually runs the AI investigation — skill-scoped graphs, hybrid RAG (Qdrant + BM25) over ingested manuals, OCR, and optional web search. Called by the backend; never called directly by the frontend.

→ [docs/architecture/workflow-agent.md](docs/architecture/workflow-agent.md)

## 5. Connectors

External systems that plug into XANA over the OpenXANA manifest contract — each a separate deployable, registered with the backend by URL. Two ship today:

- **Web / storage** — authenticated wiki/document-portal connector → [docs/architecture/connectors-web.md](docs/architecture/connectors-web.md)
- **CRM (Dynamics AX)** — Microsoft Dynamics CRM adapter, ADFS auth → [docs/architecture/connectors-crm-dynamics-ax.md](docs/architecture/connectors-crm-dynamics-ax.md)

---

## What's still pending

- **FusionPass / Process Twin integration via the Industry Fusion Data Space** — these pipelines aren't wired to a real backend yet (see "What's real vs. demo" in [Frontend architecture](docs/architecture/frontend.md)); connecting them through the Industry Fusion Data Space is planned, not done.
- **Keycloak / IFRIC Registry authentication** — today's auth is XANA's own DB-backed users with a custom HMAC-signed session token, applied inconsistently across routes (see "Auth coverage" in [Backend architecture](docs/architecture/backend.md)). Extending or replacing it with Keycloak / IFRIC Registry — covering both individual user auth and company/tenant-level identity and setup — is planned, not done.

---

For AI coding agents working in this repo, [`CLAUDE.md`](CLAUDE.md) is the maintained ground truth on module layout and cross-cutting rules.
