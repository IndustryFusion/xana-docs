# 5. Projects

> [docs](../README.md) → [Guides](README.md) → Projects
> Previous: [4. Connecting data sources](04-connectors.md) · Next: [6. Workbenches](06-workbenches.md)

A **project** is the unit a technician actually opens and works inside — it bundles together the global config from [4. Connecting data sources](04-connectors.md) into one scoped workspace. Everything a workbench does (which skill/investigation graph runs, which AI answers, which manuals it can cite) is decided by the project it belongs to, not configured per-workbench.

Projects live under the Support AI workspace: **Support → New project** (`/support/new`), a 5-step wizard.

## What differentiates one project from another

| Step | Field | What it controls |
|---|---|---|
| 1. Name | `name`, `description` | Just identification. |
| 2. Skill | `selectedSkillId` | Which LangGraph skill graph a workbench in this project runs (`workflow-agent/src/graphs/registry.py` maps a skill id to an analysis/continue/synthesis graph triple). Today's shipped skill is metal-processing support; new skills register into the same registry and become selectable here. |
| 3. MCP servers | `enabledMcpServerIds` | Which of the org's registered MCP servers ([4](04-connectors.md)) this project enables. **Not yet actually reachable by an investigation** — see the MCP note in [4](04-connectors.md) and [ARCHITECTURE.md §2](../ARCHITECTURE.md#2-connecting-to-data-sources-tool-calling-connectors-and-mcp). |
| 4. AI & Ontology | `defaultAiProviderId`, `selectedOntologyId` | Which AI provider answers for this project, and which domain-ontology graph (classes/edges the investigation consults) applies. |
| 5. Knowledge scope | `knowledgeScope` | Which folders/documents from a registered knowledge connector are in-bounds for this project's RAG retrieval — this is what keeps one project's manuals from leaking into another's answers. |

A connector itself (the CRM or knowledge source) isn't picked per-project the same explicit way — it's referenced through the knowledge scope selection and through however cases/workbenches resolve their CRM data; the project is the layer that narrows a shared connector down to *this* project's slice of it.

## Changing a project later

**Support → (a project) → Settings** lets an admin update MCP servers, AI provider, and ontology after creation. **The skill is not one of them** — Settings shows it as locked (a padlock icon, "The skill is locked after project creation. Contact an admin to change it.") with no control to change it. The backend does support changing it (`force_skill_change`, a deliberate, explicit flag precisely because it swaps the graph a project's workbenches run on) — but no frontend page sets that flag today, so doing it requires a direct API call, not a Settings-page action.

## Keeping the knowledge index current

Setting a knowledge scope doesn't mean every workbench query re-reads it live — manuals are ingested and indexed (Qdrant + BM25) once, ahead of time, and a workbench's retrieval only ever searches what's already indexed (see [ARCHITECTURE.md §6](../ARCHITECTURE.md#6-the-rag-pipeline--tying-it-together)). Each project card on the **Support** project list shows its ingestion status and a **Reingest** control (`POST /projects/:id/reingest`) — use it after changing a project's knowledge scope, or with the "restart from scratch" option after a chunking/embedding-model change, so newly added or changed manuals actually become citable. If a workbench isn't finding something you just added to the knowledge scope, reindexing — not re-analyzing — is usually the fix.

## Who can do what

Creating and configuring projects is an admin action. Once a project exists, any user with the `technician` role can open it and create workbenches inside it — see [6. Workbenches](06-workbenches.md).
