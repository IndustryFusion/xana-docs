# Connector: CRM (Dynamics AX)

> [docs](../README.md) → [Architecture](../ARCHITECTURE.md) → [Connectors](connectors-web.md) → CRM / Dynamics AX
> See also: [`connectors/CRM/dynamics-ax/README.md`](../../connectors/CRM/dynamics-ax/README.md) for the full manifest schema, endpoint list, and Kubernetes deploy.

.NET 8 / ASP.NET Core, port 8080 by default (`Dockerfile` `EXPOSE`/`ASPNETCORE_URLS`) — the root `docker-compose.yml` maps it to host port 8081 only to avoid colliding with the web connector, which also defaults to 8080. A lightweight adapter that exposes an on-premise Microsoft Dynamics CRM installation — secured with ADFS — through a REST API, implementing the OpenXANA connector contract (`contractVersion: "1.3.0"`, `Program.cs`).

## What it does

- Authenticates against ADFS via forms-based auth, caching the FedAuth/MSISAuth cookies (30-minute TTL) so the backend isn't paying an auth round-trip on every request
- Exposes Dynamics entities — accounts, cases (`incidents`), assets, product/component cards — through simplified REST endpoints on top of the Dynamics OData v8.2 API, with OData pagination (`@odata.nextLink`) handled for the caller
- Serves `GET /openxana/manifest`: the resource hierarchy, field-name mappings, and business rules (e.g. status-code labels) that let XANA use consistent field names across different CRM products without hardcoding Dynamics specifics anywhere else in the stack

## Two isolated modules in one connector

This is ASP.NET Core **minimal APIs** — routes are mapped directly in `Program.cs` (`app.MapGet(...)`, etc.), not MVC controllers.

- `Services/DynamicsService.cs` — the original Service/Support-facing entities (accounts, cases, activities, notes) that the Service & Support workbench flow reads.
- `Services/SalesService.cs` — a **separate, additive** set of routes for the Sales AI vertical's appointment data (Dynamics' Sales/Activities module, a different entity set from Service's cases). Its ADFS login/cookie-cache logic is **intentionally duplicated**, not shared with `DynamicsService.cs` — the class's own header comment is explicit that this is deliberate: "sales and support are different verticals with different owners, and this keeps a change to one service from ever being able to affect the other." This mirrors the sales/support code-isolation rule that applies on the backend and frontend too — see [7. Sales module](../guides/07-sales-module.md).

## Where to go next

- The other connector: [Web / storage](connectors-web.md)
- How an admin registers one: [4. Connecting data sources](../guides/04-connectors.md)
- What consumes CRM case data day-to-day: [Workbenches guide](../guides/06-workbenches.md); what consumes appointment data: [Sales module guide](../guides/07-sales-module.md)
