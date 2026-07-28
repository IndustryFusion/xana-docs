# 6. Workbenches

> [Documentation](../README.md) → [Guides](README.md) → Workbenches
> Previous: [5. Projects](05-projects.md) · Next: [7. Sales module](07-sales-module.md)

A **workbench** is one case a technician is working: it links a CRM account, case, and optionally a product to a scoped set of knowledge documents, and drives an AI investigation over that evidence. Everything else in Service & Support — connected data sources, AI model, domain knowledge, knowledge scope ([4](04-connectors.md), [5](05-projects.md)) — exists to make a workbench possible.

## Creating one

Open a project and you land on its dashboard, which is also the workbench list. Start a new workbench directly from there. At minimum, give it a title; optionally attach the CRM case it's for and narrow its knowledge scope if needed. Any technician can do this — no administrator step required per workbench, unlike connecting a data source or setting up the project itself.

## Running an investigation

From a workbench, asking for repair steps sends it to the investigation engine, which runs the project's selected skill: it pulls in context from the linked CRM case, retrieves relevant passages from whatever manuals are **already indexed** for the project, and generates observations, hypotheses, and troubleshooting steps with citations back to the source manuals and notes. Analyzing a case never re-reads the knowledge scope itself — that's a separate, explicit step (the project's re-index control, see [5. Projects](05-projects.md#keeping-the-knowledge-index-current)), kept out of the everyday investigation path so a case doesn't wait on a live re-read of your documents. If a workbench isn't citing a manual you just added, re-index the project first. If the investigation engine is temporarily unavailable, the workbench still gets a result — a placeholder one, clearly flagged, with no AI steps or citations — so you know to re-analyze once it's back.

What a technician does from there, all from the same workbench page:

- Walk through the generated **steps**, marking each one done, failed, skipped, or needing customer confirmation, with an optional note.
- Add **notes** and **attachments** (photos, video — read automatically) on the workbench as a whole.
- **Chat** with the copilot — for example, ask for help on a stuck step, or report that a step failed, succeeded, or was skipped — which triggers a focused follow-up response rather than a full re-analysis.
- Give **feedback** on a specific citation (outdated, incorrect, not relevant, or something else) or a copilot message — this is how bad evidence gets flagged rather than silently trusted.
- **Escalate** a workbench if it's stuck beyond what the investigation can resolve.
- **Resolve** a workbench with a summary, root cause, and the actions actually taken — or **reopen** a resolved one if the fix didn't hold.

## Language

The interface's language setting is passed through on every investigation and follow-up, so the technician's own language controls the language of generated steps and copilot replies, not a fixed setting on the deployment.

## Isolation from Sales

Workbenches, the Service & Support area, and its investigation skill are a separate vertical from Sales ([7](07-sales-module.md)) — they don't share code or data, only the same connector and AI-model infrastructure described in [4. Connecting data sources](04-connectors.md).
