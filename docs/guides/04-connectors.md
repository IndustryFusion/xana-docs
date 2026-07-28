# 4. Connecting data sources

> [Documentation](/docs/README.md) → [Guides](/docs/guides/README.md) → Connecting data sources
> Previous: [3. User management](/docs/guides/03-user-management.md) · Next: [5. Projects](/docs/guides/05-projects.md)

Everything under the admin area's **Connections** section is global, org-wide configuration — set up once by an administrator, then reused by every project. This is where you plug in *where your data and your AI come from*; a project ([5. Projects](/docs/guides/05-projects.md)) later picks which of these it actually uses.

## Connectors (CRM / knowledge)

A connector is an external system — a CRM or a document/wiki portal — registered by its address. XANA doesn't hardcode support for any particular CRM or wiki product: every connector implements the same self-describing contract, so XANA can read its structure, its fields, and its business rules once, at the time it's registered, rather than needing custom code per product.

Two connectors ship today, each a separate, independently deployable adapter, registered the same way:

- A connector for Microsoft Dynamics CRM (on-premise)
- A generic connector for authenticated wiki/document portals

Either one — or a connector you build yourself against the same contract — can be registered here. **This is the main door for extending what XANA connects to**: any system that speaks the same contract plugs in exactly like the two above do, with no changes needed anywhere else in the product. ERP is a planned future connector type, not shipped yet — it would plug in the same way once available.

## AI models

An AI model configuration is a pluggable setup: which provider, which model, and its access credentials (stored securely). If one wasn't already set up during initial setup, add one here.

This per-project configuration powers **workbench investigations** ([6. Workbenches](/docs/guides/06-workbenches.md)) — a project picks which one it uses, see [5. Projects](/docs/guides/05-projects.md). Some other AI-assisted features elsewhere in XANA (like connector setup and the sales report's executive summary) use a shared, deployment-wide default instead of a per-project choice, since they aren't tied to one specific project.

## Integrations (optional)

The admin area's Connections section also manages optional **tool integrations** — external tools and data sources a project can enable alongside its knowledge base, registered through emerging open standards for connecting AI systems to external services. Today this covers registration and configuration; making an enabled integration's tools available to an investigation's reasoning step directly is on the roadmap. See [Architecture §2](/docs/ARCHITECTURE.md#_2-connecting-to-your-data-tools-and-connectors) for more.

## Domain knowledge

Domain-knowledge graphs are managed separately, under the admin area's domain-knowledge section — these describe the structure of your domain (for example, how products and components relate to each other) that an investigation consults while reasoning through a case; they aren't part of connector setup itself. A project selects one to use.
