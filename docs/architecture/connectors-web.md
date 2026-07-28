# Connector: web / storage

> [Documentation](../README.md) → [Architecture](../ARCHITECTURE.md) → [Connectors](connectors-crm-dynamics-ax.md) → Web / storage
> See also: [service details](../../connectors/storages/web/README.md).

A configurable connector for authorized websites, wikis, and document portals — **not** a general internet crawler. Every request it makes is constrained by how it's configured: which sites it's allowed to reach, and sensible limits on size, depth, and time, so it can't be pointed somewhere it shouldn't or asked to do something unbounded.

## What it does

- Signs in to a configured site the way that site expects.
- Reads a site's folder and document structure into a consistent shape XANA can browse.
- Streams document downloads through the hub, rather than exposing the source system directly.

## How it plugs in

Like every connector, it exposes one self-describing entry point that the hub reads once, at the time it's registered — see [4. Connecting data sources](../guides/04-connectors.md). From there, its own browsing capabilities are what let a person explore a site's structure and pull manuals into a project's knowledge base.

## Security posture

Only explicitly configured sites are reachable; requests toward internal or non-public network ranges are blocked by default; credentials are never stored or logged in plain text. Which sites and credentials a deployment uses are configured at deployment time, not exposed as something changeable through the product itself.

## Where to go next

- The other connector: [CRM (Dynamics)](connectors-crm-dynamics-ax.md)
- How an admin registers one: [4. Connecting data sources](../guides/04-connectors.md)
