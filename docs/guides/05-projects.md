# 5. Projects

> [Documentation](/docs/README.md) → [Guides](/docs/guides/README.md) → Projects
> Previous: [4. Connecting data sources](/docs/guides/04-connectors.md) · Next: [6. Workbenches](/docs/guides/06-workbenches.md)

A **project** is the unit a technician actually opens and works inside — it bundles the global configuration from [4. Connecting data sources](/docs/guides/04-connectors.md) into one scoped workspace. Everything a workbench does — which skill runs, which AI answers, which manuals it can cite — is decided by the project it belongs to, not configured per workbench.

Projects live under the Service & Support workspace, set up through a short, guided wizard.

## What differentiates one project from another

| Setting | What it controls |
|---|---|
| Name | Just identification. |
| Skill | Which investigation reasoning graph a workbench in this project runs. The skill available in production today supports industrial equipment maintenance and repair; new skills become selectable here as they're added. |
| Integrations | Which of the org's registered tool integrations ([4](/docs/guides/04-connectors.md)) this project enables. As noted there, these aren't yet reachable by an investigation's reasoning step — that wiring is on the roadmap. |
| AI model & domain knowledge | Which AI model answers for this project, and which domain-knowledge graph applies. |
| Knowledge scope | Which folders and documents from a connected knowledge source are in bounds for this project's retrieval — this is what keeps one project's manuals from leaking into another's answers. |

A connector itself isn't picked per project the same explicit way — it's referenced through the knowledge-scope selection and through however a project's cases resolve their CRM data; the project is the layer that narrows a shared, org-wide connector down to *this* project's slice of it.

## Changing a project later

Project settings let an administrator update integrations, AI model, and domain knowledge after setup. The skill a project runs is locked once the project is created and isn't changeable from this screen — if you need to change it, reach out to your XANA contact.

## Keeping the knowledge index current

Setting a knowledge scope doesn't mean every workbench question re-reads it live — manuals are indexed once, ahead of time, and a workbench's retrieval only ever searches what's already indexed (see [Architecture §6](/docs/ARCHITECTURE.md#_6-the-retrieval-pipeline-tying-it-together)). Each project shows its indexing status along with a control to re-index it — use that after changing a project's knowledge scope, so newly added or changed manuals actually become citable. If a workbench isn't finding something you just added, re-indexing — not re-analyzing — is usually the fix.

## Who can do what

Setting up and configuring projects is an administrator action. Once a project exists, any technician can open it and create workbenches inside it — see [6. Workbenches](/docs/guides/06-workbenches.md).
