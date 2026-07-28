# The investigation engine

> Part of [the documentation](/docs/README.md) → [Architecture: the investigation engine](/docs/architecture/workflow-agent.md).

The investigation engine is the service that runs an AI investigation over a case: it pulls in CRM and knowledge-base context through the hub, searches your knowledge base using a combined meaning-and-wording retrieval approach, and returns cited evidence and troubleshooting steps grounded in the manuals and notes it actually found.

## What it handles beyond text search

- **Document understanding at ingestion time** — scanned or image-heavy manual pages are made readable, and diagrams get a plain-language description, so they're searchable the same way ordinary text is.
- **Technician photo and video uploads** — read at upload time, so they can be referenced later in the investigation without redoing that work.
- **Optional web search** — used only to fill a gap when your own knowledge base doesn't cover something, never as a substitute for it.

The full technical walkthrough — how the reasoning loop actually proceeds, what it can look up, and how retrieval is scored — lives in the [architecture overview](/docs/ARCHITECTURE.md).

## Where to go next

- How it fits with the rest of XANA: [Architecture: the investigation engine](/docs/architecture/workflow-agent.md)
- How it works together with the hub: [Integration](/workflow-agent/INTEGRATION.md)
