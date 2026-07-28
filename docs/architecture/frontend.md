# The interface

> [Documentation](/docs/README.md) → [Architecture](/docs/ARCHITECTURE.md) → The interface
> See also: [service details](/frontend/README.md).

**Next.js (App Router)**, listening on port 3000. It is the **only** UI, and it talks to **only** the hub — never directly to connectors, the investigation engine, or any other external system. Everything the user sees is fetched or posted through one client layer in the interface's codebase.

## Structure

| Area | What it is |
|---|---|
| Login + workspace selector | The landing page after auth, listing the AI workspaces available to the signed-in user. |
| Workspace-scoped area | Each workspace (Service & Support, Sales, ...) gets its own scoped area. A couple of illustrative, not-yet-wired surfaces (incident-style lists tied to a future process-monitoring integration) live here too, reachable only by direct URL with no nav entry point — see "What's real vs. illustrative" below. |
| Service & Support | Project list, project setup wizard, and per-project pages (workbenches, settings, ontology). |
| Sales | The weekly-report configuration page. |
| Admin | Global, org-wide configuration: connections (connectors + AI providers + integrations), users, ontologies, skills. Not workspace-scoped — one admin area serves every workspace. |
| Developer | Developer-facing tooling, a separate top-level area from the day-to-day product surfaces. |

## What's real vs. illustrative

Not every page is wired to a live backend call today. A couple of surfaces tied to a planned future process-monitoring integration (see [Still on the roadmap](/README.md#still-on-the-roadmap)) currently render illustrative data rather than a live feed — they're not reachable from any in-app navigation, only by direct URL, and are called out here rather than left for you to discover. Everything reachable from normal navigation — connectors, projects, workbenches, the sales report — is real and backed by the hub.

## Where to go next

- Using the app as a technician or admin: [the guides](/docs/guides/README.md).
- What the hub it talks to actually does: [The hub](/docs/architecture/backend.md).
