# CRM connector (Dynamics)

> Part of [the documentation](/docs/README.md) → [Architecture: CRM connector](/docs/architecture/connectors-crm-dynamics-ax.md).

A lightweight adapter that exposes an on-premise Microsoft Dynamics CRM installation — secured with ADFS — to the rest of XANA, through the same connector contract every data source uses.

## What it does

- Signs in through your existing ADFS setup, using the same forms-based authentication your CRM already expects, and keeps that session alive sensibly so the hub isn't re-authenticating on every request.
- Exposes the CRM entities Service & Support and Sales each need — accounts, cases, assets, and product/component records for Service & Support; appointments for Sales — through a simplified, consistent interface, including navigating relationships between them (for example, an account's assets, or a case's activity history).
- Handles pagination transparently, so a caller never has to think about how a large result set is split up on the CRM side.
- Publishes the same self-describing entry point every connector does, so the rest of XANA can use consistent field names and status labels without any Dynamics-specific logic living anywhere else in the product.

## Two isolated capabilities in one connector

This connector serves both Service & Support and Sales, and keeps that separation down to its own internals: Service & Support–facing entities and Sales' appointment-data entities are two distinct, additive extensions, with their own sign-in and session handling kept intentionally separate — so a change made for one workspace can never affect the other. See [Sales module](/docs/guides/07-sales-module.md) and [CRM connector architecture](/docs/architecture/connectors-crm-dynamics-ax.md) for more.

## Where to go next

- How it fits with the rest of XANA: [Architecture: CRM connector](/docs/architecture/connectors-crm-dynamics-ax.md)
- How an admin registers one: [4. Connecting data sources](/docs/guides/04-connectors.md)
