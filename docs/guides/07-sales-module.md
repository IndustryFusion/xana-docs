# 7. Sales module

> [Documentation](/docs/README.md) → [Guides](/docs/guides/README.md) → Sales module
> Previous: [6. Workbenches](/docs/guides/06-workbenches.md)

**Sales** is a separate workspace from Service & Support — a different purpose, kept isolated at every level. It reuses the same CRM connector and AI-model infrastructure from [4. Connecting data sources](/docs/guides/04-connectors.md), but nothing else from Service & Support or workbenches.

## What it does today: the automated weekly sales report

A phase-one deliverable: every Friday, on a fixed schedule, XANA:

1. Reads the connected CRM's appointment data — a separate slice of your CRM from the case data Service & Support reads — for the past week, for a configured roster of sales managers.
2. Categorizes appointments against a configured set of appointment types.
3. Computes week-over-week trends and flags accounts whose pipeline stage has been stuck for an extended period.
4. Generates an AI executive summary of the week alongside the mechanical stats, using the deployment's shared default AI model rather than a per-project one, since there's no Sales "project" to configure one for.
5. Renders a bilingual report as a PDF and emails it to a configured list of recipients.

## Configuring it

From the Sales workspace's configuration page, administrators can set:

- **CRM connector** — which registered CRM connector to read from.
- **Recipients** — who receives the emailed report.
- **Managers** — the roster of sales managers to cover.
- **Test report** — run the pipeline on demand, without waiting for the next scheduled run, useful for checking a configuration change before it goes out for real.

There's no separate "sales" role gating this — see [3. User management](/docs/guides/03-user-management.md) — access is through the workspace, the same as Service & Support.

## Email delivery

Sending the report by email is configured separately from the rest of a deployment, since it depends on your organization's own email setup rather than anything else XANA needs. Everything else the report needs — recipients, manager roster, categories, which CRM connector to use — is regular project-style configuration. Without email delivery configured, report generation still works; only the send step won't.

## Why it's isolated

Sales code is kept from ever touching Service & Support's code or data, and vice versa — shared infrastructure (like the underlying connector framework and general account/database plumbing) is reused only in ways that add to it, never in ways that change how Service & Support already depends on it. That isolation goes all the way down to the CRM connector itself: the Sales-specific extension to the CRM connector is intentionally kept separate from the Service & Support–facing one, including its own sign-in handling, so a change to one can never affect the other (see [CRM connector architecture](/docs/architecture/connectors-crm-dynamics-ax.md)).
