# Backend

> [docs](../README.md) → [Architecture](../ARCHITECTURE.md) → Backend
> See also: [`backend/README.md`](../../backend/README.md) for MongoDB setup, migration, and running tests.

NestJS, port 4000. This is the API hub every other piece of XANA goes through — the frontend talks to nothing else, the workflow-agent calls back into it for CRM/knowledge data rather than hitting connectors directly, and it's the only service with durable state (MongoDB).

## Modules (`backend/src/`)

| Module | Purpose |
|---|---|
| `auth` | DB-backed users with hashed passwords and role-gated CRUD; custom HMAC-signed session tokens. |
| `users` | User CRUD, roles, temporary passwords, last-admin protection. |
| `connectors` | Registers CRM/knowledge connectors via the OpenXANA manifest contract; AI-assisted field mapping. |
| `proxy` | HTTP proxy to connector base URLs, with a TTL-based response cache. |
| `knowledge` | Browse connector folders, download PDF/DOCX content. |
| `support` | Cases, accounts, per-case product mappings — the Service & Support vertical's data layer. |
| `workbenches` | Workbench CRUD; bridges to the workflow-agent for analyze/continue. |
| `sales` | The Sales AI vertical — appointment reads, report building, AI insights, PDF rendering, email delivery, the weekly cron. Isolated from `support`/`workbenches`. |
| `projects` | Ties a skill, AI provider, ontology, MCP servers, and knowledge scope together into one project; also owns the per-project knowledge reindex flow (`POST /projects/:id/reingest`, surfaced as the reingest control on each project card in `frontend/app/support/page.tsx`). |
| `skills` | Catalog of available LangGraph skills (workflow-agent's `registry.py`). |
| `ontology` | Domain-ontology graphs (classes/edges) a project can select. |
| `ai-providers` | Pluggable per-project LLM provider config, encrypted API key. Resolved per project (`AiProvidersService.resolveForProject`) and passed through to the workflow-agent on every analyze/continue call — **but not used for connector field mapping or sales insights**, see the `ai` module below. |
| `ai` | A separate, simpler LLM helper (`AiService.completeJson`) that always calls the single global `OPENAI_COMPATIBLE_*` env config directly — no per-project resolution. This is what connector field-mapping actually calls, and what the sales module's AI executive summary calls (there's no sales "project" to resolve a per-project `ai-providers` entry from). Don't assume "AI provider" always means the `ai-providers` store — check which of these two a given feature actually imports. |
| `mcp-servers` | Generic OAuth2/PKCE MCP client registration. Configuration only today — see the honest note in [ARCHITECTURE.md §2](../ARCHITECTURE.md#2-connecting-to-data-sources-tool-calling-connectors-and-mcp) on why an enabled MCP server isn't yet callable by an investigation. |
| `migrations` | One-time startup seeding (`startup-migration.service.ts`), not an ongoing migration tool: base ontology, an `ai-providers` entry from `OPENAI_COMPATIBLE_*` env vars, and the initial admin user (see [2. First login](../guides/02-first-login.md)) — each step is a no-op once its target collection already has data. |
| `developer` | An internal debug-tooling surface backing `frontend/app/developer`. |
| `database` | Mongo connection/schemas, legacy JSON→Mongo migration. |

Persistence is MongoDB — see `backend/src/database/schemas/` for the full schema list (11 collections as of this writing).

## Why it's the hub, not a thin proxy

The backend, not the frontend or the workflow-agent, owns the OpenXANA manifest resolution, the connector response cache, and role/session enforcement. Both the frontend and the workflow-agent trust it to have already resolved "which field on this CRM means what" — neither re-implements that logic.

## Where to go next

- What calls into it: [Frontend](frontend.md)
- What it calls out to for AI: [Workflow agent](workflow-agent.md)
- What external systems it proxies to: [Connectors](connectors-web.md), [CRM connector](connectors-crm-dynamics-ax.md)
