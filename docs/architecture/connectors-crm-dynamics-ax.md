# Connector: CRM (Dynamics)

> [Documentation](../README.md) → [Architecture](../ARCHITECTURE.md) → [Connectors](connectors-web.md) → CRM / Dynamics
> See also: [service details](../../connectors/CRM/dynamics-ax/README.md).

**.NET 8 / ASP.NET Core**, its own deployable, listening on port 8080 by default (the root Docker Compose setup maps it to host port 8081 to avoid colliding with the web connector, which also defaults to 8080). Built with ASP.NET Core's minimal-API style rather than MVC controllers. A lightweight adapter that exposes an on-premise Microsoft Dynamics CRM installation — secured with **ADFS** — through a REST API implementing the OpenXANA connector contract (`contractVersion: "1.3.0"`).

## What it does

- Authenticates against ADFS via forms-based auth, caching the resulting `FedAuth`/`MSISAuth` session cookies for roughly 30 minutes so the hub isn't paying an auth round-trip on every request.
- Exposes Dynamics entities — accounts, cases (Dynamics calls them incidents), assets, and product/component cards — through simplified REST endpoints on top of the **Dynamics OData v8.2** API, with OData `@odata.nextLink`-style pagination handled transparently for the caller.
- Serves `GET /openxana/manifest`: the resource hierarchy, field-name mappings, and business rules (e.g. status-code labels) that let XANA use consistent field names across different CRM products without hardcoding Dynamics specifics anywhere else in the stack.

## The manifest contract

This is the piece that matters if you're building a connector for a different CRM: the manifest is a JSON document describing every resource the connector exposes, in a shape any OpenXANA client (XANA's hub, or your own) can consume without hardcoding endpoints.

Each resource entry describes:

- **A list operation** — HTTP method, path, where in the response body the result array lives, where the pagination continuation token lives, and the query parameters it accepts (e.g. a page-size default).
- **A detail operation** — path (with path parameters mapped to resource fields) for fetching one record by id.
- **Field mappings** — a plain object translating OpenXANA's standard field names (`id`, `name`, `status`, ...) to this CRM's actual field names (`accountid`, `statuscode`, ...), so a client never needs CRM-specific knowledge.
- **Children** — nested resources reachable from a parent (an account's assets, a case's activities/notes/owner), each either a list or a single object.
- **Rules** (optional) — how to interpret a field's raw values, e.g. which numeric status codes count as active vs. closed, and their human-readable labels.

For this adapter specifically, the exposed resources are accounts (with asset, product-card, and case children), cases (with activity, note, and owner children), and assets (with component-card children) — the set Service & Support's workbench flow reads day to day.

## Two isolated capabilities in one connector

This connector serves two workspaces that are deliberately kept separate everywhere else in XANA — Service & Support and Sales — and it carries that separation down to its own internals:

- The original Service & Support–facing entities (accounts, cases, activity history) that a workbench investigation reads.
- A **separate, additive** set of routes for the Sales workspace's appointment data, covering a different part of Dynamics entirely (its Sales/Activities module rather than Service's case module). Its own ADFS sign-in and session-caching logic is **intentionally duplicated** rather than shared, specifically so a change made for one workspace can never affect the other. This mirrors the sales/support isolation described in [7. Sales module](../guides/07-sales-module.md).

## Where to go next

- The other connector: [Web / storage](connectors-web.md)
- How an admin registers one: [4. Connecting data sources](../guides/04-connectors.md)
- What consumes CRM case data day to day: [Workbenches guide](../guides/06-workbenches.md); what consumes appointment data: [Sales module guide](../guides/07-sales-module.md)
