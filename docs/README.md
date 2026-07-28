# Documentation

> [Root README](../README.md) → Documentation

## Guides — how to use XANA

Sequential, start-to-finish: setup → first login → users → connectors → projects → workbenches → sales. See [guides/](guides/README.md).

## Architecture — how XANA is built

- [ARCHITECTURE.md](ARCHITECTURE.md) — the technical deep-dive: the LangGraph workflow agent core, tool calling, LLM/embedding/OCR configuration, the RAG pipeline, and how the frontend/backend fit around it. Verified against source.
- One page per service — what it does and how it fits with the rest:
  - [architecture/frontend.md](architecture/frontend.md)
  - [architecture/backend.md](architecture/backend.md)
  - [architecture/workflow-agent.md](architecture/workflow-agent.md)
  - [architecture/connectors-web.md](architecture/connectors-web.md)
  - [architecture/connectors-crm-dynamics-ax.md](architecture/connectors-crm-dynamics-ax.md)
- [setup/docker-deployment.md](setup/docker-deployment.md) — Docker Compose profiles, ports, volumes, troubleshooting.
- [setup/kubernetes-deployment.md](setup/kubernetes-deployment.md) — how the separate [GitOpsRepo](https://github.com/IndustryFusion/GitOpsRepo) Helm/Fleet setup deploys these same services to a cluster.

## Service-level references

Each architecture page above links to its service's own README for operational detail (local run, build, tests) — those stay next to the code they describe:

| Service | Doc |
|---|---|
| Backend | [`backend/README.md`](../backend/README.md) — MongoDB collections, migration, running tests |
| Workflow agent | [`workflow-agent/README.md`](../workflow-agent/README.md) — RAG stack, OCR, web search, API endpoints |
| Workflow agent ↔ backend wiring | [`workflow-agent/INTEGRATION.md`](../workflow-agent/INTEGRATION.md) |
| Dynamics CRM connector | [`connectors/CRM/dynamics-ax/README.md`](../connectors/CRM/dynamics-ax/README.md) — OpenXANA manifest, endpoints, Kubernetes deploy |
| Web/storage connector | [`connectors/storages/web/README.md`](../connectors/storages/web/README.md) — connector config format, security defaults |
