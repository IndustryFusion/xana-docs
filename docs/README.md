# Documentation

> [Root README](../README.md) → Documentation

## Guides — how to use XANA

Sequential, start to finish: setup → first login → users → connectors → projects → workbenches → sales. See [the guides](guides/README.md).

## Architecture — how XANA is built

- [Architecture overview](ARCHITECTURE.md) — the technical deep-dive: how the investigation engine reasons about a case, how the underlying AI model is configured, the retrieval pipeline behind every citation, and how the interface and hub fit around it.
- One page per service — what it does and how it fits with the rest:
  - [The interface](architecture/frontend.md)
  - [The hub](architecture/backend.md)
  - [The investigation engine](architecture/workflow-agent.md)
  - [Connector: web / storage](architecture/connectors-web.md)
  - [Connector: CRM (Dynamics)](architecture/connectors-crm-dynamics-ax.md)
- [Running it locally](setup/docker-deployment.md) — the local/single-host path.
- [Running it in production](setup/kubernetes-deployment.md) — how a real deployment is structured and kept up to date.

## Service-level details

Each architecture page above links to a short page on that service for a closer look at what it's responsible for day to day:

| Service | Page |
|---|---|
| The hub | [Service details](../backend/README.md) |
| The interface | [Service details](../frontend/README.md) |
| The investigation engine | [Service details](../workflow-agent/README.md) |
| How the engine and the hub work together | [Integration details](../workflow-agent/INTEGRATION.md) |
| CRM connector | [Service details](../connectors/CRM/dynamics-ax/README.md) |
| Web / storage connector | [Service details](../connectors/storages/web/README.md) |
