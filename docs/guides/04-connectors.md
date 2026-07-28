# 4. Connecting data sources

> [docs](../README.md) → [Guides](README.md) → Connecting data sources
> Previous: [3. User management](03-user-management.md) · Next: [5. Projects](05-projects.md)

Everything under **Admin → Connections** (`/admin/connections`) is global, org-wide configuration — done once by an admin, then reused by every project. This is where you plug in *where the data and the AI come from*; a project ([5. Projects](05-projects.md)) later picks which of these it uses.

## Connectors (CRM / knowledge)

A connector is an external system — a CRM or a document/wiki portal — registered by base URL. XANA doesn't hardcode any particular CRM or wiki product: every connector implements the **OpenXANA manifest contract** (`GET {baseUrl}/openxana/manifest`), a self-describing API — resources, fields, hierarchy, business rules — that XANA reads once at registration time.

**There is no dedicated "Connectors" screen or "Add connector" button** — despite the backend having a fully-built `POST /connectors` endpoint for exactly that, no page in the frontend calls it. Registering one today goes through the **MCP Servers** panel instead:

1. Admin → Connections → **MCP Servers → Add MCP Server**.
2. Give it a name, then change **Category** from its default (`tooling`) to `crm` or `knowledge`. The form collapses to just a base URL field, and the submit button relabels itself **Add Connector**.
3. Submitting calls `POST /mcp-servers`, not `POST /connectors` — the backend's `mcp-servers.service.ts`, specifically for the `crm`/`knowledge` category branch, fetches `/openxana/manifest` from that URL and saves it into the connectors store, *while also* creating an MCP-server record for it. That's why a connector you register this way then shows up in the **MCP Servers** list, not a separate one — there isn't one.
4. Field mapping isn't a step in this flow — it happens lazily, the first time something actually reads data through the connector (`support.service.ts`), not at registration time.

This repo ships two working connectors as examples/production adapters, each a separate deployable registered the same way:

- `connectors/CRM/dynamics-ax/` — Microsoft Dynamics CRM (on-prem), via ADFS auth
- `connectors/storages/web/` — generic authenticated wiki/document-portal connector

Either one — or a connector you build yourself against the same manifest contract — can be registered here.

## AI providers

An AI provider is a pluggable LLM configuration: provider type (OpenAI-compatible, Anthropic, Azure OpenAI, Ollama), base URL, model, and an API key (stored encrypted, via `APP_ENCRYPTION_KEY`). If you set `OPENAI_COMPATIBLE_*` in `.env`, one is seeded automatically on first startup; otherwise add one here.

This per-project `ai-providers` store is used for **workbench AI investigation** ([6. Workbenches](06-workbenches.md)) — a project picks which one it uses, see [5. Projects](05-projects.md). It is **not** what powers connector field mapping (above) or the [sales module](07-sales-module.md)'s AI executive summary — those two call a separate, simpler helper that always uses the single global `OPENAI_COMPATIBLE_*` env config directly, regardless of any per-project provider you add here. See [Backend architecture](../architecture/backend.md) for why.

## MCP servers (optional)

Admin → Connections also manages **MCP servers** — external tool/data sources reachable over MCP (OAuth2/PKCE-authenticated), beyond the CRM/knowledge connector contract. A project can enable one or more MCP servers alongside its knowledge scope, **but as of this writing an enabled MCP server's tools are not actually offered to the investigation's reasoning LLM yet** — the connection/registration layer is built, the wire into the tool-calling loop isn't. See [ARCHITECTURE.md §2](../ARCHITECTURE.md#_2-connecting-to-data-sources-tool-calling-connectors-and-mcp) for the verified detail.

## Ontologies

Domain-ontology graphs (classes/edges) are managed separately under **Admin → Ontologies** — these describe the structure of the domain (e.g. product/component relationships) that the investigation graph consults; they aren't part of connector registration itself. A project selects one ontology to use.
