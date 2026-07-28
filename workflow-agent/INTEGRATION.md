# How the engine and the hub work together

> Part of [the documentation](/docs/README.md) → [Architecture: the investigation engine](/docs/architecture/workflow-agent.md).

The investigation engine and the hub are two separate services with a simple relationship: the hub calls the engine to start or continue an investigation, the engine calls back into the hub for whatever CRM or knowledge data it needs, and the result is handed back to the interface as part of the same workbench a technician is already looking at — nothing changes from the technician's point of view.

## What each side is responsible for

- **The hub** decides *when* to call the engine (a technician asking for repair steps, or continuing a conversation), and is responsible for turning the result into the workbench record the interface displays.
- **The investigation engine** decides *how* to investigate — which reasoning graph to run, what to look up, and how to ground its answer in evidence — and never talks to a connected system directly; it always asks the hub.

## If the engine is unavailable

If the investigation engine is disabled or temporarily unreachable, the hub doesn't fail the request — it returns a placeholder result, clearly flagged as incomplete, so the technician knows to retry once the engine is back rather than seeing a broken workbench.

## Where to go next

- The reasoning loop itself: [Architecture overview](/docs/ARCHITECTURE.md)
- The day-to-day flow this powers: [Workbenches guide](/docs/guides/06-workbenches.md)
