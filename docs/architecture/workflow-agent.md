# The investigation engine

> [Documentation](../README.md) → [Architecture](../ARCHITECTURE.md) → The investigation engine
> See also: [service details](../../workflow-agent/README.md) and [how the engine and the hub work together](../../workflow-agent/INTEGRATION.md).

This is where an investigation actually happens. It never talks to a connected system directly — it asks the hub for CRM and knowledge data, since the hub has already resolved what a field in your system actually means.

## Reasoning is scoped by skill

XANA doesn't run one general-purpose reasoning pipeline for every domain — each domain has its own dedicated reasoning graph, selected per project (see [5. Projects](../guides/05-projects.md)). The skill shipped in production today supports industrial equipment maintenance and repair; new domains are added as new skills over time, without disturbing the ones already running.

**The full technical walkthrough of this service lives in the main architecture document, not here** — it's the core the whole product is built around, so it gets the detailed treatment there:

- [§1 — The core: the investigation engine](../ARCHITECTURE.md#_1-the-core-the-investigation-engine) — how an investigation actually proceeds, and how a follow-up conversation shares the same reasoning core
- [§2 — Connecting to your data](../ARCHITECTURE.md#_2-connecting-to-your-data-tools-and-connectors) — what the reasoning step can look up, and how far third-party tool integrations reach today
- [§3 — Bringing your own AI model](../ARCHITECTURE.md#_3-bringing-your-own-ai-model) · [§4 — How your knowledge base becomes searchable](../ARCHITECTURE.md#_4-how-your-knowledge-base-becomes-searchable) · [§5 — Reading manuals, photos, and diagrams](../ARCHITECTURE.md#_5-reading-manuals-photos-and-diagrams) · [§6 — The retrieval pipeline](../ARCHITECTURE.md#_6-the-retrieval-pipeline-tying-it-together)

## What it exposes

At a high level, this service offers a small set of capabilities to the rest of XANA: running a new investigation, continuing one in response to a chat message or step update, summarizing how a case was ultimately resolved, indexing (or re-indexing) a project's knowledge base, and processing an uploaded photo or video so it can be referenced during an investigation. Everything else about it — the reasoning loop, the retrieval pipeline, the AI model it calls — is covered in the [architecture overview](../ARCHITECTURE.md).

## Observability

Every investigation — its lookups, its retrieval, its reasoning steps — is fully traceable end to end, which is what makes an unexpected answer something you can actually diagnose rather than a black box.

## Where to go next

- Who calls it, and how the result is used: [The hub](backend.md), [Workbenches guide](../guides/06-workbenches.md)
