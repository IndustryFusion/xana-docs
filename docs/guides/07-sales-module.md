# 7. Sales module

> [docs](../README.md) → [Guides](README.md) → Sales module
> Previous: [6. Workbenches](06-workbenches.md)

**Sales AI** (`/sales`) is a separate workspace vertical from Support AI — a different purpose, different backend module (`backend/src/sales/`), kept isolated at the code level. It reuses the same CRM connector and AI-provider infrastructure from [4. Connecting data sources](04-connectors.md), but nothing else from Support/workbenches.

## What it does today: automated Weekly Sales Report

A phase-1 deliverable: every Friday at 13:00 Europe/Berlin (ahead of the stakeholder's 14:00 Berlin deadline), the backend:

1. Queries the configured CRM connector's Sales/Appointments data (a separate entity from the Service/case data Support reads — its own isolated `Services/SalesService.cs` in `connectors/CRM/dynamics-ax/`, routes mapped in `Program.cs`) for the past week's appointments belonging to a configured roster of sales managers.
2. Categorizes appointments by matching their subject against a configured set of exact title prefixes (e.g. "Bestandskundenbesuch", "Vorführung (B)").
3. Computes trend deltas (this week vs. trailing 4-week average) and flags stalled-pipeline accounts stuck at a non-final stage for 21+ days.
4. Generates an AI executive summary of the week alongside the mechanical stats — via the same global, env-configured LLM helper used for connector field mapping ([4](04-connectors.md)), **not** the per-project `ai-providers` store, since there's no sales "project" to resolve one from.
5. Renders a bilingual (German + English) PDF report and emails it to a configured recipient list via the Gmail API.

## Configuring it

**Sales → (config page)** (`/sales`), admin-facing:

- **CRM connector** — pick which registered CRM connector to query.
- **Recipients** — who receives the emailed report.
- **Managers** — the roster of sales managers to cover (name + CRM email; resolved to a CRM system-user at run time, not a hardcoded id).
- **Test report** — run the pipeline on demand (dry-run, or actually send) without waiting for Friday, useful for verifying config changes before the next scheduled run.

There's no separate "sales" user role gating this — see [3. User management](03-user-management.md) — access is via the workspace, same as Support AI.

## Email setup (separate from the main `.env`)

Gmail-API delivery needs its own config, loaded from `backend/.env.sales` (copy from `backend/.env.sales.example`), not the main `backend/.env`: a Google Workspace service-account key file (`SALES_GOOGLE_SERVICE_ACCOUNT_KEY_FILE`, domain-wide delegation with the `https://mail.google.com/` scope) plus `SALES_GOOGLE_IMPERSONATE_SUBJECT` (the mailbox the service account authenticates as) and `SALES_MAIL_FROM` (the visible sender, which can differ if it's a configured "Send As" alias). Everything else the report needs — recipients, manager roster, category prefixes, CRM connector id — lives in MongoDB (the config page above), not in this file. Without it, PDF generation and the AI summary still work; only the actual email send will fail.

## Why it's isolated

Per this repo's sales/support isolation rule, sales code must not touch `backend/src/support/`, `backend/src/workbenches/`, `frontend/app/support/`, or the workflow-agent's support skill, and vice versa. Shared infra (`ProxyService`/`ConnectorsService`, generic Mongo/auth infra) is reused only additively — the sales-specific CRM extension lives entirely in its own `Services/SalesService.cs`, which goes further than just not editing the support-facing connector code: its ADFS login/cookie logic is **intentionally duplicated** rather than extracted into anything shared, so a change to one can never affect the other (see [CRM connector architecture](../architecture/connectors-crm-dynamics-ax.md)).
