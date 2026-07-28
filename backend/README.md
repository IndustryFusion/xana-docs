# The hub

> Part of [the documentation](../docs/README.md) → [Architecture: the hub](../docs/architecture/backend.md).

The hub is XANA's central API and system of record — the piece every other service goes through, and the only one with durable, lasting state.

## What lives here

Everything that needs to persist across sessions is stored centrally and owned by the hub: people and their roles, connected CRM and knowledge systems and how their fields translate into a consistent shape, projects and their configuration, workbenches and their full investigation history, and the Sales workspace's report configuration and history.

Short-lived, in-memory housekeeping (like response caching for a connector) lives alongside it but isn't part of that durable record.

## Bringing forward existing data

If a deployment is moving from an earlier, file-based way of storing this data into the database-backed hub, that migration is a one-time, explicit step — it never overwrites data that's already there, and running it again after it's already succeeded is a safe no-op.

## Where to go next

- How it fits with the rest of XANA: [Architecture: the hub](../docs/architecture/backend.md)
- What calls into it: [The interface](../docs/architecture/frontend.md)
- What it calls out to for AI: [The investigation engine](../docs/architecture/workflow-agent.md)
