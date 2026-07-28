# 3. User management

> [docs](../README.md) → [Guides](README.md) → User management
> Previous: [2. First login](02-first-login.md) · Next: [4. Connecting data sources](04-connectors.md)

Users are managed at **Admin → Users** (`/admin/users`), admin-only.

## Roles

Each user has one or more roles: `admin`, `technician`. A user can hold both.

- **admin** — manages connectors, AI providers, MCP servers, ontologies, users, and projects (the setup side of XANA).
- **technician** — works inside projects: opens workbenches, runs investigations, walks through repair steps.

There is no separate "sales" role — access to the Sales AI workspace isn't gated by a distinct role in the same way; see [7. Sales module](07-sales-module.md).

## Adding a user

1. Admin → Users → **Add user**.
2. Fill in first name, last name, username, at least one role, and a **temporary password** (min. 8 characters).
3. The new user must change that temporary password on their first login — this is enforced server-side (`mustChangePassword` is always set `true` for admin-created accounts), separately from the one-time env-seeded admin from [2. First login](02-first-login.md), which is exempt.

Share the username + temporary password with the person out of band; they'll be prompted to set their own password the first time they log in.

## Editing, resetting, removing

- **Edit** changes name, username, or roles.
- **Reset password** sets a new temporary password and forces a change on next login — use this if someone is locked out.
- **Remove** deletes the account.

**Last-admin protection:** the backend refuses to demote or remove the last remaining admin account. If you need to hand off admin duties, promote another user to `admin` first, then remove or demote the old one.
