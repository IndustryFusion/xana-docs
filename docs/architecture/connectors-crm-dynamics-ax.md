# Connector: CRM (Dynamics)

> [Documentation](../README.md) → [Architecture](../ARCHITECTURE.md) → [Connectors](connectors-web.md) → CRM / Dynamics
> See also: [service details](../../connectors/CRM/dynamics-ax/README.md).

A lightweight adapter that exposes an on-premise Microsoft Dynamics CRM installation — secured with ADFS — to the rest of XANA through the same connector contract every data source uses.

## What it does

- Signs in through your existing ADFS setup, and keeps that session alive sensibly so the hub isn't re-authenticating on every request.
- Exposes the CRM entities Service & Support and Sales each need — accounts, cases, assets and product records for Service & Support; appointments for Sales — through a simplified, consistent interface.
- Publishes the same self-describing entry point every connector does: the resource structure, field-name translations, and business rules (like status labels) that let XANA use consistent terms across different CRM products without hardcoding anything specific to Dynamics elsewhere in the stack.

## Two isolated capabilities in one connector

This connector serves two workspaces that are deliberately kept separate everywhere else in XANA — Service & Support and Sales — and it carries that same separation down to its own code:

- The original Service & Support–facing entities (accounts, cases, activity history) that a workbench investigation reads.
- A **separate, additive** set of routes for the Sales workspace's appointment data, covering a different part of Dynamics entirely. Its own sign-in and session handling is **intentionally kept separate** rather than shared, precisely so a change made for one workspace can never affect the other. This mirrors the sales/support isolation described in [7. Sales module](../guides/07-sales-module.md).

## Where to go next

- The other connector: [Web / storage](connectors-web.md)
- How an admin registers one: [4. Connecting data sources](../guides/04-connectors.md)
- What consumes CRM case data day to day: [Workbenches guide](../guides/06-workbenches.md); what consumes appointment data: [Sales module guide](../guides/07-sales-module.md)
