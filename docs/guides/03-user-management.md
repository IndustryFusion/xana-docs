# 3. User management

> [Documentation](/docs/README.md) → [Guides](/docs/guides/README.md) → User management
> Previous: [2. First login](/docs/guides/02-first-login.md) · Next: [4. Connecting data sources](/docs/guides/04-connectors.md)

Users are managed from the admin area, by administrators only.

## Roles

Each person has one or more roles:

- **Administrator** — manages connected data sources, AI models, integrations, domain knowledge, people, and projects (the setup side of XANA).
- **Technician** — works inside projects: opens workbenches, runs investigations, walks through repair steps.

A person can hold both. There is no separate "sales" role — access to the Sales workspace isn't gated the same explicit way; see [7. Sales module](/docs/guides/07-sales-module.md).

## Adding a person

1. From the admin area, add a new user.
2. Fill in their name, username, at least one role, and a temporary password.
3. They'll be required to set their own password the first time they sign in — the one exception is the very first, initial administrator account from [2. First login](/docs/guides/02-first-login.md), which isn't required to.

Share the username and temporary password out of band; they'll be prompted to choose their own password on first sign-in.

## Editing, resetting, removing

- **Edit** changes a person's name, username, or roles.
- **Reset password** issues a new temporary password and requires a change on next sign-in — use this if someone is locked out.
- **Remove** deletes the account.

**There's always at least one administrator.** XANA won't let you demote or remove the last remaining administrator account — promote someone else to administrator first if you need to hand off that responsibility.
