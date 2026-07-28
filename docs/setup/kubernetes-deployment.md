# Kubernetes deployment

> [docs](../README.md) → Kubernetes deployment
> See also: [Docker deployment](docker-deployment.md) (local/single-host) · [root README](../../README.md)

**The Kubernetes manifests are not in this repository.** `xana-business` is the application monorepo (the five services described in the root README); how those services get deployed to a cluster is a separate concern, owned by a separate, dedicated deployment repo: **[GitOpsRepo](https://github.com/IndustryFusion/GitOpsRepo)**. If you have it checked out, it sits as a sibling of this repo (`../GitOpsRepo`, i.e. `xana-dev/GitOpsRepo` next to `xana-dev/xana-business`) — that's also the fixed relative path XANA's own build/deploy tooling assumes.

This split is deliberate GitOps practice: application code changes (this repo) and the deployed state of the cluster (GitOpsRepo) are reviewed, versioned, and rolled back independently.

## Concepts, briefly

- **Helm chart** — a templated bundle of Kubernetes YAML (Deployment, Service, ConfigMap, Secret, etc.) for one workload, parameterized by a `values.yaml`.
- **Rancher Fleet / Continuous Delivery** — the GitOps controller: it watches a Git repo (GitOpsRepo) and continuously reconciles the cluster to match what's committed there, instead of anyone running `kubectl apply`/`helm install` by hand against production.
- **Namespace** — all XANA workloads deploy into one namespace, `xana`.

## How GitOpsRepo is structured

```
GitOpsRepo/
└── helm/
    └── charts/
        ├── mongo/                in-cluster MongoDB
        ├── qdrant/                in-cluster vector store
        ├── backend/                NestJS API (+ optional connector-seed Job)
        ├── frontend/                Next.js UI
        ├── workflow-agent/        LangGraph / FastAPI agent
        ├── web-connector/          web/wiki knowledge connector
        ├── dynamics-connector/      Dynamics CRM connector
        ├── ollama/                  in-cluster Ollama (vision captioning, option A)
        └── ovms/                    in-cluster OpenVINO Model Server (vision captioning, option B)
```

One self-contained Helm chart + Fleet bundle per deployable workload — each with its own `Chart.yaml`, `values.yaml`, `fleet.yaml`, `templates/`. Every chart shares the `xana` namespace and gives its Service a **fixed, non release-prefixed name** (`mongo`, `qdrant`, `backend`, ...), so charts reach each other over stable in-cluster DNS no matter what Helm release name Fleet happens to assign — e.g. the backend chart's default `config.mongodbUri` is `mongodb://mongo:27017/xana-business`, matching the same `MONGODB_URI` shape used locally (see [1. Setup](../guides/01-setup.md)).

### Rancher GitRepo configuration

In Rancher's Continuous Delivery UI, the GitRepo's **Paths** field must list every chart folder explicitly — Fleet does not glob-expand a parent directory into multiple bundles:

```
helm/charts/mongo
helm/charts/qdrant
helm/charts/backend
helm/charts/frontend
helm/charts/workflow-agent
helm/charts/web-connector
helm/charts/dynamics-connector
helm/charts/ollama
helm/charts/ovms
```

Each path becomes its own Bundle (`<gitrepo-name>-<last-path-segment>`, e.g. `xana-business-backend`). Don't add a blank/root path — there's no chart at the repo root, and Fleet creates a pointless empty bundle for it.

Deploy order doesn't matter for a bare install (`envFrom` only needs the ConfigMap/Secret to *exist*, not the pods behind it to be Ready), but for a fully working stack, bring datastores up first: `mongo`, `qdrant` → `backend`, `workflow-agent` → `frontend` → `web-connector` / `dynamics-connector` (only if those connectors are in use). `ollama` and/or `ovms` can deploy independently at any point — workflow-agent only calls whichever one `config.ollamaBaseUrl` points at, and only when `config.visionCaptionEnabled` is `"true"`.

## Secrets

**As currently deployed, secrets are not managed through Helm values at all** — this is a real gap between what the charts' comments still suggest and what actually renders. `backend`, `workflow-agent`, `dynamics-connector`, and `qdrant` each have a `templates/secret.yaml`, but every one of them is wrapped in `{{- if false }}` and disabled, with the chart's own comment explaining why: letting Fleet reconcile a rendered `Secret` from `values.yaml` would reset it to empty placeholders on every sync, so real secrets are instead set **directly on the cluster**, out-of-band from Helm/Fleet entirely.

Concretely, for a target cluster:

1. Create each service's Secret object by hand — the exact keys and Secret name a given deployment expects are documented in that chart's own (disabled) `templates/secret.yaml`, and the deployment's `secretRef` names the Secret it looks for. Use `kubectl create secret generic <name> -n xana --from-literal=KEY=value ...` with those.
2. **Protect it from Fleet before the next sync**, or Fleet will delete it as an object no longer part of the rendered chart: `kubectl annotate secret <name> helm.sh/resource-policy=keep -n xana --overwrite`. Already-running pods keep their env vars regardless; only a future rollout/restart would fail to start if the Secret got deleted out from under it.
3. Repeat for `backend`, `workflow-agent`, `dynamics-connector`, and `qdrant` — the four charts with a (disabled) `secret.yaml` — for whichever are in use.

`web-connector` is different again — it has no `secret.yaml`/Secret at all. Its wiki credentials are baked into the Docker image at **build time** via `docker build --build-arg WIKI_USERNAME=... --build-arg WIKI_PASSWORD=...` (see `connectors/storages/web/Dockerfile`), not supplied at deploy time through Kubernetes/Helm in any form.

`frontend` has no secrets to manage.

**Never commit real credentials to GitOpsRepo** — `values.yaml`'s (now-unused) placeholder secret fields exist only as chart documentation of what a Secret would need, not a real input path today.

## Images

`backend`, `frontend`, `workflow-agent`, `web-connector`, and `dynamics-connector` are custom-built images (`imageRegistry` + `image` in each chart's `values.yaml`; default registry `ibn40/`). `mongo`, `qdrant`, `ollama`, and `ovms` use public upstream images and are untouched by the image-tag-patching flow below.

Building/pushing those five images from this monorepo and patching the resulting tags into GitOpsRepo's Helm `values.yaml` files is one pipeline — in this environment, that's the `xana-build-deploy` workflow: it builds the changed service(s), pushes to Docker Hub under `ibn40/`, and patches GitOpsRepo's chart values, but it deliberately stops short of committing/pushing GitOpsRepo itself — that step stays a manual, reviewed action.

## Manual install (no Fleet)

```bash
helm install mongo ./helm/charts/mongo -n xana --create-namespace
helm install qdrant ./helm/charts/qdrant -n xana
# Create backend-secret by hand first — see "Secrets" above — then:
helm install backend ./helm/charts/backend -n xana
helm install frontend ./helm/charts/frontend -n xana
# Create workflow-agent-secret by hand first, same pattern, then:
helm install workflow-agent ./helm/charts/workflow-agent -n xana
# optional connectors:
# web-connector's credentials are build-time (see "Secrets" above) — no secret needed here:
helm install web-connector ./helm/charts/web-connector -n xana
# create dynamics-connector-secret by hand first, then:
helm install dynamics-connector ./helm/charts/dynamics-connector -n xana
# vision-captioning backend — pick one (workflow-agent's config.ollamaBaseUrl decides which is used):
helm install ollama ./helm/charts/ollama -n xana
helm install ovms ./helm/charts/ovms -n xana
```

Run from inside a `GitOpsRepo` checkout. This is the same chart set Fleet installs — useful for a one-off cluster or debugging a chart before wiring it into Continuous Delivery. `--set secrets.x=...` is **not** a working shortcut for any of these charts today — see "Secrets" above for why, and the real per-chart Secret-name/keys to create instead.

## Where to go next

- What each of these charts actually runs: [Frontend](../architecture/frontend.md) · [Backend](../architecture/backend.md) · [Workflow agent](../architecture/workflow-agent.md) · [Connectors](../architecture/connectors-web.md)
- Local, non-Kubernetes deployment: [Docker deployment](docker-deployment.md)
