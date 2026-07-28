# 2. First login

> [docs](../README.md) → [Guides](README.md) → First login
> Previous: [1. Setup](01-setup.md) · Next: [3. User management](03-user-management.md)

## Log in as the seeded admin

On first backend startup, if no users exist yet in MongoDB, XANA seeds exactly one admin account from your `.env`:

- Username: `APP_ADMIN_USERNAME` (default `admin`)
- Password: `APP_ADMIN_PASSWORD` (default `admin123`)

This seeding only ever happens once — it's a one-time bootstrap, not an ongoing auth source. Once any user exists, changing `APP_ADMIN_*` in `.env` has no further effect. This account does **not** need a forced password change on first login (only users created later via the admin panel do — see [3. User management](03-user-management.md)); change the password yourself if you're not on a throwaway local instance.

Open http://localhost:3000 and log in with those credentials.

## What you land on

After login you see the **workspace selector** — XANA is split into separate verticals, each its own top-level area:

| Workspace | Route | Purpose |
|---|---|---|
| Support AI | `/support` | Field-technician case support: workbenches, AI investigation, repair steps. See [6. Workbenches](06-workbenches.md). |
| Sales AI | `/sales` | Sales-team automation (currently: the weekly sales report). See [7. Sales module](07-sales-module.md). |

Admins also see a link into the **Admin** area (`/admin`), which is not workspace-scoped — it configures things every workspace depends on.

## First actions as admin

Before anyone can do useful work, an admin needs to, in order:

1. **Register connectors** — point XANA at your CRM and/or knowledge sources. See [4. Connecting data sources](04-connectors.md).
2. **Add an AI provider** (if you didn't already set `OPENAI_COMPATIBLE_*` in `.env`) — needed for AI-assisted connector field mapping and for workbench/report AI features. Same guide.
3. **Add the people who'll actually use it** — see [3. User management](03-user-management.md).
4. **Create a project** — a project is what ties a connector, an AI provider, a knowledge scope, and a skill together into something a technician can open a workbench against. See [5. Projects](05-projects.md).

Once a project exists, non-admin users (technicians) can open it and start creating workbenches — no further admin action needed per case.
