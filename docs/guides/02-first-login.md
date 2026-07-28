# 2. First login

> [Documentation](../README.md) → [Guides](README.md) → First login
> Previous: [1. Setup](01-setup.md) · Next: [3. User management](03-user-management.md)

## Signing in as the initial administrator

The very first time a new deployment starts up, with no accounts yet on record, XANA creates exactly one administrator account from the credentials provided during setup (see [1. Setup](01-setup.md)).

This only ever happens once — it's a one-time bootstrap, not an ongoing source of truth. Once any account exists, that initial setup no longer has any effect. Change that first password promptly once you're past initial setup, the same way you would for any admin account.

## What you land on

After signing in you see the **workspace selector** — XANA is split into separate verticals, each its own top-level area:

| Workspace | Purpose |
|---|---|
| Service & Support | Field-technician case support: workbenches, AI investigation, repair steps. See [6. Workbenches](06-workbenches.md). |
| Sales | Sales-team automation (currently: the weekly sales report). See [7. Sales module](07-sales-module.md). |

Administrators also see a link into the **Admin** area, which isn't scoped to one workspace — it configures things every workspace depends on.

## First actions as an administrator

Before anyone can do useful work, an administrator needs to, in order:

1. **Connect your data sources** — point XANA at your CRM and/or knowledge base. See [4. Connecting data sources](04-connectors.md).
2. **Add an AI model** (if one wasn't already set up during initial setup) — needed for AI-assisted setup and for workbench/report AI features. Same guide.
3. **Add the people who'll actually use it** — see [3. User management](03-user-management.md).
4. **Set up a project** — a project ties a connected data source, an AI model, a knowledge scope, and a skill together into something a technician can open a workbench against. See [5. Projects](05-projects.md).

Once a project exists, technicians can open it and start creating workbenches — no further admin action is needed per case.
