# The hub

> [Documentation](../README.md) → [Architecture](../ARCHITECTURE.md) → The hub
> See also: [service details](../../backend/README.md).

**NestJS**, listening on port 4000, the API hub every other piece of XANA goes through. The interface talks to nothing else; the investigation engine calls back into it for CRM/knowledge data rather than hitting connectors directly; and it's the only service with durable state, persisted in **MongoDB**.

## Modules

| Module | Responsibility |
|---|---|
| Auth | Database-backed users with hashed passwords and role-gated CRUD; sessions use a signed session token rather than a third-party identity provider (see [Still on the roadmap](../../README.md#still-on-the-roadmap)). |
| Users | User CRUD, roles, temporary passwords, last-admin protection. |
| Connectors | Registers CRM/knowledge connectors via the OpenXANA manifest contract; AI-assisted field mapping. |
| Proxy | HTTP proxy to connector base URLs, with a TTL-based response cache to avoid re-hitting a connector for the same request. |
| Knowledge | Browses connector folders, downloads PDF/DOCX content for ingestion. |
| Support | Cases, accounts, per-case product mappings — the Service & Support vertical's data layer. |
| Workbenches | Workbench CRUD; bridges to the investigation engine for analyze/continue calls. |
| Sales | The Sales vertical — appointment reads, report building, AI insights, PDF rendering, email delivery, the weekly schedule. Isolated from Support/Workbenches at the code level. |
| Projects | Ties a skill, AI provider, ontology, integrations, and a knowledge scope together into one project; owns the per-project knowledge-reindex flow surfaced as the **Reingest** control on each project card. |
| Skills | Catalog of available investigation skills (the engine's skill registry, mirrored here for the setup UI). |
| Ontology | Domain-ontology graphs (classes/edges) a project can select — consulted by the investigation graph's context-enrichment step. |
| AI providers | Pluggable per-project AI model configuration, encrypted API key. Resolved per project and passed through to the engine on every analyze/continue call — but **not** used for connector field mapping or the sales AI summary, which call a separate, simpler global LLM helper that always uses one deployment-wide model configuration. Don't assume "AI provider" always means the per-project store; check which of these two a given feature actually calls. |
| Integrations (MCP) | Generic OAuth2/PKCE-authenticated MCP server registration. Configuration only today — see the honest note in [ARCHITECTURE.md §2](../ARCHITECTURE.md#_2-connecting-to-your-data-tools-and-connectors) on why an enabled integration isn't yet callable by an investigation. |
| Migrations | One-time startup seeding, not an ongoing migration tool: the base ontology, an initial AI-provider entry from environment configuration, and the initial admin account — each step is a no-op once its target collection already has data. |
| Developer | An internal debug-tooling surface backing the interface's developer area. |
| Database | Mongo connection/schemas, and a one-time legacy-file→Mongo migration path for deployments upgrading from an earlier, file-based version. |

Durable state spans 11 MongoDB collections as of this writing — workbenches, connectors, connector response cache, case/support state, connector field mappings, users, projects, skills, AI providers, integrations, and domain ontologies.

## Why it's the hub, not a thin proxy

The hub — not the interface, not the investigation engine — owns OpenXANA manifest resolution, the connector response cache, and role/session enforcement. Both the interface and the engine trust that it has already resolved "which field on this CRM means what"; neither re-implements that logic.

## Where to go next

- What calls into it: [The interface](frontend.md)
- What it calls out to for AI: [The investigation engine](workflow-agent.md)
- What external systems it proxies to: [Connectors](connectors-web.md), [CRM connector](connectors-crm-dynamics-ax.md)
