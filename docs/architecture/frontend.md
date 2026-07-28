# Frontend

> [docs](../README.md) → [Architecture](../ARCHITECTURE.md) → Frontend
> See also: [`frontend/README.md`](../../frontend/README.md) for local run/build commands.

Next.js (App Router), port 3000. It is the **only** UI, and it talks to **only** the NestJS backend — never directly to connectors, the workflow-agent, or any external system. Everything the user sees is fetched or posted through one client module, [`frontend/lib/api.ts`](../../frontend/lib/api.ts).

## Structure

| Path | What it is |
|---|---|
| `app/page.tsx` | Login + the workspace selector — the landing page after auth, listing the available AI workspaces (`WORKSPACES` in `lib/types.ts`). |
| `app/[workspace]/...` | The workspace-scoped area. Its workbench sub-paths redirect into `app/support/...`; `fusionpass` and `process-twin` render their (mock) content directly here instead, with no nav entry point anywhere in the app — reachable only by direct URL. (The notification bell drawer shows the same mock incident data inline in its own overlay — it never links to these two page routes, so it isn't an alternate path to them.) |
| `app/support/` | The Service & Support workspace: project list, project creation wizard (`support/new`), and per-project pages (`support/[projectId]/{workbenches,settings,ontology}`). |
| `app/sales/` | The Sales AI workspace: the weekly-report configuration page. |
| `app/admin/` | Global, org-wide configuration: connections (connectors + AI providers + MCP servers), users, ontologies, skills. Not workspace-scoped — one admin area serves every workspace. |
| `app/developer/` | Developer-facing tooling, separate top-level area. |

## What's real vs. demo

Per the honest assessment in [ARCHITECTURE.md](../ARCHITECTURE.md), not every page is wired to a live backend call — some surfaces (e.g. FusionPass/Process Twin incident lists) are still mock/demo-only. Check `lib/api.ts` and the page in question before assuming an affordance is real.

## Where to go next

- Using the app as a technician or admin: [guides](../guides/README.md).
- What the backend it talks to actually does: [Backend](backend.md).
