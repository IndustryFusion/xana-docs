# Running it locally

> Part of [the documentation](/docs/README.md) → [Architecture](/docs/ARCHITECTURE.md). See also the [root README](/README.md).

Run the entire product from the repository root via Docker Compose — closer to a production topology than the hot-reload native dev flow, and the fastest way to see XANA end to end without provisioning a cluster.

## Core stack

The default Compose profile brings up the interface, the hub, MongoDB, the investigation engine, and Qdrant.

| Service | URL |
|---|---|
| Interface | `http://localhost:3000` |
| Hub API | `http://localhost:4000` |
| Hub health | `http://localhost:4000/health` |
| Investigation engine health | `http://localhost:8000/health` |
| MongoDB | `localhost:27017` |
| Qdrant | `http://localhost:6333` |

The initial administrator account is seeded from configuration on first startup (see [2. First login](/docs/guides/02-first-login.md)) — always set a real password for it before anything beyond a throwaway local instance.

### Full stack with connectors

A second Compose profile adds the two example connectors, for anyone who wants to see a connected data source working rather than an empty product:

| Connector | Host URL | Internal URL (hub → connector) |
|---|---|---|
| Web / knowledge connector | `http://localhost:8080` | `http://web-connector:8080` |
| Dynamics CRM connector | `http://localhost:8081` | `http://dynamics-connector:8080` |

Enabling connector auto-seeding registers both connectors for the default workspace in MongoDB on first startup, so you don't have to register them by hand to try the connector flow end to end.

## Profiles

| Profile | Services |
|---|---|
| *(default)* | MongoDB, Qdrant, hub, interface, investigation engine |
| Connectors | Web connector, Dynamics connector, one-shot connector-seeding job |

## Environment

Key variables (see the repo root's environment template for the full list):

| Variable | Purpose |
|---|---|
| `NEXT_PUBLIC_API_URL` | Browser → hub base URL (keep it matching the hub's published port) |
| `CORS_ORIGINS` | Origins the hub accepts requests from — must include the interface's origin |
| `OPENAI_COMPATIBLE_*` | AI model endpoint/key for workbench AI and connector field mapping |
| `CRM_*` | Dynamics connector credentials |
| `WIKI_USERNAME` / `WIKI_PASSWORD` | Substituted into the mounted web-connector configuration |
| `SEED_CONNECTORS` | Whether the one-shot seeding job registers the example connectors automatically (`true` by default) |

## Volumes

| Volume | Purpose |
|---|---|
| `mongo_data` | Durable hub state |
| `qdrant_data` | Vector store |
| `backend_uploads` | Workbench attachment uploads |
| `workflow_agent_data` | Retrieval-pipeline cache |

## Investigation-engine-only development

To iterate on the investigation engine plus Qdrant without the rest of the stack, its own Compose file brings up just that pair, expecting the hub reachable on the host at port 4000.

## Troubleshooting

- **Interface can't reach the hub** — confirm `NEXT_PUBLIC_API_URL` points at a reachable hub, and rebuild the interface since any change to that setting.
- **CORS errors in the browser** — confirm `CORS_ORIGINS` includes the interface's origin.
- **Connectors profile fails to start** — confirm `CRM_URL`/`CRM_USERNAME`/`CRM_PASSWORD` are set.
- **AI-powered analysis fails** — confirm `OPENAI_COMPATIBLE_*` is set and reachable, then restart the investigation engine.
- **Skip connector auto-seeding** — set `SEED_CONNECTORS=false`.

## Where to go next

- What each of these services actually does: [The interface](/docs/architecture/frontend.md) · [The hub](/docs/architecture/backend.md) · [The investigation engine](/docs/architecture/workflow-agent.md) · [Connectors](/docs/architecture/connectors-web.md)
- Production deployment: [Running it in production](/docs/setup/kubernetes-deployment.md)
