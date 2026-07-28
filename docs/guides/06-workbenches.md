# 6. Workbenches

> [docs](../README.md) → [Guides](README.md) → Workbenches
> Previous: [5. Projects](05-projects.md) · Next: [7. Sales module](07-sales-module.md)

A **workbench** is one case a technician is working: it links a CRM account/case/incident (and optionally a product, and a process-twin incident) to a scoped set of knowledge documents, and drives an AI investigation over that evidence. Everything else in Support AI — connectors, AI provider, ontology, knowledge scope ([4](04-connectors.md), [5](05-projects.md)) — exists to make a workbench possible.

## Creating one

Open a project and you land on its **Dashboard**, which is also the workbench list — there's no separate "Workbenches" section to navigate into first (an older dedicated route was merged into the Dashboard; it now just redirects there). Click **New workbench** directly from it. At minimum give it a title; optionally attach the CRM account/case/incident it's for and a knowledge scope override (if narrower than the project's default). Any user with the `technician` role can do this — no admin step required per workbench, unlike registering a connector or creating the project itself.

## Running an investigation

From the workbench detail page, **Get repair steps** (or **re-analyze**) sends the workbench to the workflow-agent, which runs the project's selected skill's LangGraph investigation loop: it enriches context from the linked CRM case, retrieves relevant passages from whatever manuals are **already indexed** for the project (hybrid RAG — vector + BM25 + rerank), builds an evidence map, and generates observations, hypotheses, and troubleshooting steps with citations back to the source manuals/notes. Analyze does **not** re-resolve or re-ingest the knowledge scope itself — that's a separate, explicit step (the project's **Reingest** control, see [5. Projects](05-projects.md#keeping-the-knowledge-index-current)), kept deliberately out of the analyze/continue path to avoid live connector round-trips and timeouts on every single call. If a workbench isn't citing a manual you just added, reindex the project first. If the workflow-agent is disabled or unreachable, the workbench still gets a result, but a placeholder one — no AI steps or citations — flagged so you know to re-analyze once the agent is back.

What a technician does from there, all from the same workbench page:

- Walk through the generated **steps**, marking each `done`, `failed`, `skipped`, or `needs_customer_confirmation`, with an optional note.
- Add **notes**/**attachments** (photos, video — OCR'd automatically) on the workbench as a whole. (A per-step "add observation" API and its UI strings exist in the codebase, but nothing currently calls it — the "observations" you'll actually see in the investigation panel are the AI's own, generated during analysis, not technician-authored.)
- **Chat** with the copilot (`step_chat`) — e.g. ask for help on a stuck step, or report a step failed/done/skipped — which re-invokes the agent's *continue* graph for a follow-up response, rather than a full re-analysis.
- Give **feedback** on a specific citation (outdated/incorrect/not relevant/other) or a copilot message (correct/incorrect) — this is how bad evidence gets flagged, not silently trusted.
- **Escalate** a workbench if it's stuck beyond what the investigation can resolve.
- **Resolve** a workbench with a summary, root cause, and the actions actually taken — or **reopen** a resolved one if it turns out the fix didn't hold.

## Language

The frontend's EN/DE language toggle is passed through on every analyze/continue call, overriding the workflow-agent's default — so the technician's UI language controls the language of generated steps and copilot replies, not a fixed server setting.

## Isolation from Sales

Workbenches, the `support` frontend area, and the metal-processing-support skill are a separate vertical from Sales AI ([7](07-sales-module.md)) — they don't share code or data, only the same connector/AI-provider infrastructure described in [4. Connecting data sources](04-connectors.md).
