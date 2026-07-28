# XANA Business

Most companies already have what they need to solve a case: a CRM full of history, and a shelf of manuals nobody has time to search. XANA sits on top of both — CRM and knowledge sources today, with ERP as a planned future connector type — and turns them into an AI **workspace** — a place where a person opens a case, asks XANA to investigate, and gets back a step-by-step answer with a citation for every claim, back to the exact CRM note or manual page it came from. No black-box answers: if XANA can't point to where it read something, it doesn't say it.

XANA is organized as a set of **workspaces**, one per part of the business:

- **Service & Support** — the workspace in production use today. Connect a CRM and a knowledge base, open a **workbench** for a case, and let XANA's investigation walk a technician through an evidence-backed repair, citing the CRM notes and manuals it drew from.
- **Sales** — just getting started, with an automated weekly sales report built from CRM appointment data.
- **Operations** — on the roadmap.
- **Academy** — a longer-term idea, not built yet.

Inside a workspace, an admin sets up one or more **projects** — each one scoping a connected data source, an AI model, a knowledge base, and (for Service & Support) which investigation skill it runs. From there, day-to-day users open **workbenches** — one per task. In Service & Support, a workbench is tied to a CRM case, and that's what the investigation runs against.

## Contents

*(kept up to date as documentation is added — see the [full documentation index](/docs/README.md))*

- [How XANA runs](#how-xana-runs)
- [Deploying to a real environment](#deploying-to-a-real-environment)
- [1. How XANA is built](#_1-how-xana-is-built)
- [2. The interface](#_2-the-interface)
- [3. The hub](#_3-the-hub)
- [4. The investigation engine](#_4-the-investigation-engine)
- [5. Connectors](#_5-connectors)
- [Still on the roadmap](#still-on-the-roadmap)
- [Guides — using XANA](/docs/guides/README.md) — setup → first login → users → connectors → projects → workbenches → sales module
- [Deployment](/docs/setup/docker-deployment.md)

## How XANA runs

XANA is a handful of small, independent services — an interface, a central hub, an investigation engine, and one adapter per external system it connects to — that run together as one product. For a quick local look, the whole stack can be brought up on a single machine; for a real deployment, each service runs as its own container, scaled and managed independently. See [Deployment](/docs/setup/docker-deployment.md) for both paths and what you'll need to configure first.

## Deploying to a real environment

Beyond a local look, XANA runs on Kubernetes, with each service deployed and upgraded independently and the cluster's state managed through Git rather than by hand — so a deployment is reviewable and reversible the same way a code change is.

The cluster configuration itself lives outside this repository, in a dedicated deployment repo, kept deliberately separate from the application code here: what the product *is* and how it's *currently running* are reviewed on their own timelines.

→ [Deployment](/docs/setup/kubernetes-deployment.md) for the shape of that setup and what to expect from it.

---

Below is a tour of the five things that make up XANA — what each one does and how it fits with the rest. Each point links to its full write-up in the documentation.

## 1. How XANA is built

The technical core: the investigation engine's reasoning loop, how it decides what to look up as it works a case, how the underlying AI model is configured, and the retrieval pipeline that grounds every answer in a citation instead of a guess.

→ [Architecture overview](/docs/ARCHITECTURE.md)

## 2. The interface

The web application — the only thing anyone using XANA ever opens, and the only piece that talks to the hub directly. This is where the workspace selector, the Service & Support project and workbench screens, the Sales report configuration, and the admin area all live.

→ [Interface architecture](/docs/architecture/frontend.md)

## 3. The hub

The central API and system of record. Everything durable — people, projects, connected data sources, workbenches, sales reports, AI configuration — lives here. It's the only piece that talks to connected systems and to the investigation engine, so nothing else has to.

→ [Hub architecture](/docs/architecture/backend.md)

## 4. The investigation engine

The service that actually runs an investigation: skill-specific reasoning graphs, a hybrid retrieval pipeline over your ingested manuals, document understanding (including scanned pages and diagrams), and optional web search when your own knowledge base doesn't have the answer. It's called by the hub — it's never reachable directly.

→ [Investigation engine architecture](/docs/architecture/workflow-agent.md)

## 5. Connectors

External systems plug into XANA through a shared, self-describing contract — each one a separate, independently deployable adapter, registered with the hub by address. Two ship today:

- **Web / document storage** — an authenticated connector for wikis and document portals → [Connector: web / storage](/docs/architecture/connectors-web.md)
- **CRM (Microsoft Dynamics)** — a connector for Microsoft Dynamics-based CRM systems → [Connector: CRM (Dynamics)](/docs/architecture/connectors-crm-dynamics-ax.md)

Building a new one against the same contract is how XANA connects to a data source it doesn't ship an adapter for today.

---

## Still on the roadmap

- **Deeper integration with connected process/incident data** — some workspace surfaces show illustrative data today rather than a live feed; wiring them to a real, shared data layer is planned. See "What's real vs. illustrative" in [Interface architecture](/docs/architecture/frontend.md).
- **Enterprise identity and single sign-on** — XANA manages its own users today; extending that to your company's existing identity provider, for both individual sign-in and organization-level setup, is planned.

---
