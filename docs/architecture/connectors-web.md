# Connector: storage / web

> [docs](../README.md) → [Architecture](../ARCHITECTURE.md) → [Connectors](connectors-crm-dynamics-ax.md) → Web / storage
> See also: [`connectors/storages/web/README.md`](../../connectors/storages/web/README.md) for config format, endpoints, and Docker.

NestJS, port 8080 (example). A configurable REST connector for authorized websites, wikis, and document portals — **not** a general internet crawler. Every remote request is constrained by connector config: allowed hosts, URL validation, SSRF checks, timeouts, crawl limits, file-size limits.

## What it does

- Authenticates against a configured site (form/cookie/token session)
- Parses HTML folder pages into structured JSON (folders, files, pages)
- Streams validated document downloads through the backend

## OpenXANA contract

Like every connector, it exposes `GET /openxana/manifest` as its entry point — this is what the XANA backend fetches once at registration time, per [4. Connecting data sources](../guides/04-connectors.md). Beyond the manifest, its UI-browsing endpoints (`/web-connectors/:id/folders`, `.../documents/:id/download`, etc.) are what the backend's `knowledge` module calls to let a technician browse and pull manuals into a project's knowledge scope.

## Security posture

Configured `allowedHosts` only; private/link-local/metadata-service IP ranges blocked by default; plaintext credentials rejected (env placeholders or secret refs only); cookies/tokens kept in memory and redacted from logs. Connector configs (which sites, which credentials) are deployment inputs — mounted via Docker volume or Kubernetes ConfigMap, not a runtime API resource.

## Where to go next

- The other connector: [CRM (Dynamics AX)](connectors-crm-dynamics-ax.md)
- How an admin registers one: [4. Connecting data sources](../guides/04-connectors.md)
