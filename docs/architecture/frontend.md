# The interface

> [Documentation](../README.md) → [Architecture](../ARCHITECTURE.md) → The interface
> See also: [service details](../../frontend/README.md) for local build/run notes.

The web application — the **only** thing anyone using XANA opens, and the only piece that talks to the hub directly. Nothing here ever reaches a connected system, the investigation engine, or any other external service on its own.

## Structure

| Area | What it is |
|---|---|
| Login and workspace selector | The landing page after signing in — lists the AI workspaces available to you. |
| Workspace-scoped areas | Each workspace (Service & Support, Sales, ...) has its own space for the projects and workbenches inside it. Some illustrative surfaces described below live here too. |
| Service & Support | The project list, project setup, and per-project pages — workbenches, settings, domain knowledge. |
| Sales | The weekly-report configuration page. |
| Admin | Global, org-wide configuration: connected systems, AI configuration, integrations, people, domain knowledge, skills. Not scoped to one workspace — one admin area serves all of them. |
| Developer | Internal tooling, kept separate from the day-to-day product areas. |

## What's real vs. illustrative

Not every screen is wired to a live backend today — a couple of workspace surfaces (incident-style lists tied to future process-monitoring integrations) currently show illustrative data rather than a live feed, ahead of that integration being completed. Everything described in the [Guides](../guides/README.md) — connectors, projects, workbenches, the sales report — is real and backed by the hub.

## Where to go next

- Using the app as a technician or admin: [the guides](../guides/README.md).
- What the hub it talks to actually does: [The hub](backend.md).
