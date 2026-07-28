# Running it locally

> Part of [the documentation](../README.md) → [Architecture](../ARCHITECTURE.md). See also the [root README](../../README.md).

For a quick look at XANA — a demo, an evaluation, or local development — the whole stack (interface, hub, investigation engine, database, and connectors) can run together on a single machine using standard container tooling. This is the fastest way to see XANA end to end without provisioning a cluster.

## What you get

A single-machine run brings up the full product: the interface, the hub, the investigation engine, the database, and the retrieval index, all wired together and ready to use as soon as an administrator finishes the setup steps in [1. Setup](../guides/01-setup.md).

Optionally, the same setup can bring up XANA's example connectors — a CRM connector and a knowledge-source connector — pre-registered and ready to configure, if you want to see a connected data source working rather than just the empty product.

## Getting started

Local, single-machine deployment needs container tooling installed, and the same handful of things every deployment needs (see [1. Setup](../guides/01-setup.md)): an AI model to point at, an initial administrator, and — if you're using the example connectors — credentials for whatever CRM or knowledge system you're connecting.

Beyond a quick local look, [Running it in production](kubernetes-deployment.md) covers how XANA runs as a real, scaled deployment.

## Troubleshooting

- **The interface can't reach the hub** — double check that the address the interface is configured to use actually points at a reachable hub, and that the interface has been rebuilt since any change to that setting.
- **Cross-origin errors in the browser** — the hub needs to know which origin the interface is served from; confirm that's configured to match.
- **A connector fails to start** — confirm its credentials are set correctly for whichever CRM or knowledge system it's pointed at.
- **AI-powered features fail** — confirm an AI model is configured and reachable (see [4. Connecting data sources](../guides/04-connectors.md)), then retry.

## Where to go next

- What each of these services actually does: [The interface](../architecture/frontend.md) · [The hub](../architecture/backend.md) · [The investigation engine](../architecture/workflow-agent.md) · [Connectors](../architecture/connectors-web.md)
- Production deployment: [Running it in production](kubernetes-deployment.md)
