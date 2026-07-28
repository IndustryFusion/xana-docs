# Connector: web / storage

> [Documentation](../README.md) → [Architecture](../ARCHITECTURE.md) → [Connectors](connectors-crm-dynamics-ax.md) → Web / storage
> See also: [service details](../../connectors/storages/web/README.md).

**NestJS**, its own deployable, listening on port 8080 by default. A configurable REST connector for authorized websites, wikis, and document portals — **not** a general internet crawler. Every remote request is constrained by connector configuration: allowed hosts, URL validation, SSRF checks, timeouts, crawl limits, and file-size limits.

## What it does

- Authenticates against a configured site (form, cookie, or token session) and caches the resulting session.
- Parses HTML folder pages into structured JSON — folders, files, and pages — via configurable selectors and rules (which links are folders vs. files, which to ignore).
- Streams validated document downloads through the hub rather than exposing the source system directly to the interface.

## OpenXANA contract

Like every connector, it exposes `GET /openxana/manifest` as its entry point — the self-describing manifest the hub fetches once at registration time (see [4. Connecting data sources](../guides/04-connectors.md)). Beyond the manifest, its own browsing endpoints are what the hub's knowledge module calls to let a technician browse and pull manuals into a project's knowledge scope:

| Endpoint | Purpose |
|---|---|
| `GET /web-connectors` | Lists configured connector summaries, so a caller can choose a portal |
| `GET /web-connectors/:id/folders` | Fetches the configured document root's folders, pages, and files |
| `GET /web-connectors/:id/folders/:folderId` | Lists a selected folder's direct children |
| `GET /web-connectors/:id/folders/:folderId/tree` | Returns a bounded recursive tree from a selected folder |
| `GET /web-connectors/:id/documents/:documentId/metadata` | Returns display metadata for a selected file |
| `GET /web-connectors/:id/documents/:documentId/download` | Streams a selected file through the hub |

The interface never calls these directly — it goes through the hub, and every decoded folder/document reference is re-validated against the connector's allowlist and SSRF rules before anything is fetched.

Connector configuration (which sites, which credentials) is a deployment input — mounted as a file at deploy time, not a runtime API resource — so changing which sites a connector can reach requires a configuration change and restart, not a product-level action.

## Security posture

- Only explicitly configured hosts can be reached — nothing else.
- Private, link-local, multicast, and cloud metadata-service IP ranges are blocked by default; plain `http://` is disabled unless explicitly allowed, and reaching a private network address requires two independent opt-ins (connector-level and service-level), not one.
- Plaintext credentials in configuration are rejected — only environment placeholders or secret references are accepted.
- Session cookies/tokens are kept in memory only and redacted from logs and responses.
- Recursive folder crawling enforces a max depth, a max page count, and a request timeout; downloads enforce a max file size and stream rather than buffering the full file in memory.

## Where to go next

- The other connector: [CRM (Dynamics)](connectors-crm-dynamics-ax.md)
- How an admin registers one: [4. Connecting data sources](../guides/04-connectors.md)
