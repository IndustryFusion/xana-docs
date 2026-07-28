# The hub

> [Documentation](../README.md) → [Architecture](../ARCHITECTURE.md) → The hub
> See also: [service details](../../backend/README.md) for a closer look at what it's responsible for.

This is the central API every other piece of XANA goes through — the interface talks to nothing else, the investigation engine calls back into it for CRM and knowledge lookups rather than reaching connected systems directly, and it's the only piece with durable, lasting state.

## What it's responsible for

| Area | What it covers |
|---|---|
| People and access | Accounts, roles, sessions — who can configure XANA versus who works cases. |
| Connected systems | Registering CRM and knowledge connectors, and translating each one's own field names into a consistent shape the rest of XANA can rely on. |
| Case data | Cases, accounts, and how they map to products — the data layer behind Service & Support. |
| Workbenches | Creating and managing workbenches, and handing them off to the investigation engine when it's time to analyze or continue a case. |
| Sales | The Sales workspace: appointment data, report building, AI-generated summaries, PDF rendering, and delivery — kept isolated from Service & Support end to end. |
| Projects | Tying a skill, an AI model, domain knowledge, integrations, and a knowledge scope together into one project a technician can work inside — including keeping that project's knowledge base up to date on request. |
| Skills, domain knowledge, and integrations | The catalog of available investigation skills, the domain-knowledge graphs a project can select, and registered third-party tool integrations. |
| AI configuration | Per-project AI model configuration, stored securely. |
| Setup | One-time initial setup on first startup — seeding the base domain knowledge, an initial AI configuration, and the first admin account (see [2. First login](../guides/02-first-login.md)) — each a no-op once already done. |

## Why it's the hub, not a thin pass-through

The hub — not the interface, not the investigation engine — owns figuring out what a field in your connected CRM or knowledge system actually means, and caches responses from those systems sensibly. Both the interface and the investigation engine trust that translation has already happened; neither re-implements it.

## Where to go next

- What calls into it: [The interface](frontend.md)
- What it calls out to for AI: [The investigation engine](workflow-agent.md)
- What external systems it connects to: [Connectors](connectors-web.md), [CRM connector](connectors-crm-dynamics-ax.md)
