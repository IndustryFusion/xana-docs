# Running it in production

> [Documentation](../README.md) → Running it in production
> See also: [Running it locally](docker-deployment.md) (local/single-host) · [root README](../../README.md)

**The Kubernetes manifests are not in this repository.** `xana-business` is the application monorepo (the services described in the [root README](../../README.md)); how those services get deployed to a cluster is a separate concern, owned by a dedicated deployment repository: **[GitOpsRepo](https://github.com/IndustryFusion/GitOpsRepo)**. Conventionally checked out as a sibling of this repo — that's also the fixed relative path XANA's own build/deploy tooling assumes. Application code changes (this repo) and the deployed state of the cluster (GitOpsRepo) are versioned, reviewed, and rolled back independently — deliberate GitOps practice, not an oversight.

## Concepts, briefly

- **Helm chart** — a templated bundle of Kubernetes objects (Deployment, Service, ConfigMap, Secret, ...) for one workload, parameterized by its own values file.
- **GitOps continuous delivery** (Rancher Fleet, in this deployment) — a controller that watches GitOpsRepo and continuously reconciles the cluster to match what's committed there, instead of anyone running `kubectl apply` / `helm install` by hand against production.
- **Namespace** — all XANA workloads deploy into one namespace, `xana`.

## Chart layout

One self-contained Helm chart per deployable workload, each with its own chart metadata, values file, and templates:

```
mongo                   in-cluster MongoDB
qdrant                  in-cluster vector store
backend                 the hub (NestJS) — optional connector-seed Job
frontend                the interface (Next.js)
workflow-agent          the investigation engine (LangGraph / FastAPI)
web-connector           web/wiki knowledge connector
dynamics-connector      Dynamics CRM connector
ollama                  in-cluster Ollama — vision captioning, option A
ovms                    in-cluster OpenVINO Model Server — vision captioning, option B
```

Every chart shares the `xana` namespace and gives its Service a **fixed, non-release-prefixed name** (`mongo`, `qdrant`, `backend`, ...), so charts reach each other over stable in-cluster DNS regardless of what Helm release name Fleet assigns — e.g. the hub's default MongoDB connection string is `mongodb://mongo:27017/xana-business`, matching the same connection-string shape used in [local development](docker-deployment.md).

In Rancher's Continuous Delivery UI, the GitRepo's **Paths** field must list every chart folder explicitly — Fleet doesn't glob-expand a parent directory into multiple bundles. Each path becomes its own bundle (named `<gitrepo-name>-<last-path-segment>`, e.g. `xana-business-backend`), so adding a new chart means registering its path too, not just adding the folder.

**Deploy order** isn't strict for a bare install (a Deployment's environment references only need the ConfigMap/Secret to *exist*, not the workload behind it to be Ready) — but for a stack that actually works end to end: datastores first (`mongo`, `qdrant`), then `backend` and `workflow-agent`, then `frontend`, then the connector charts you're using. `ollama`/`ovms` can deploy at any point — the investigation engine only calls whichever one its vision-captioning configuration points at, and only when that feature is turned on.

## Secrets

Secrets are deliberately **not** reconciled through the GitOps flow the way ordinary configuration is: letting Fleet render a Secret object from a values file would mean either committing real credentials to GitOpsRepo, or having Fleet reset a hand-provisioned Secret back to an empty placeholder on every sync. Instead, the four charts with real credentials to manage — `backend`, `workflow-agent`, `dynamics-connector`, and `qdrant` — each expect a Kubernetes Secret to already exist in the namespace, referenced by name from the chart's Deployment. That Secret is created directly on the cluster, out-of-band from Helm/Fleet entirely, and then protected from the next GitOps sync deleting it:

```
kubectl create secret generic <chart's-expected-secret-name> -n xana \
  --from-literal=<KEY>=<value> ...
kubectl annotate secret <chart's-expected-secret-name> -n xana \
  helm.sh/resource-policy=keep --overwrite
```

Already-running pods keep their environment regardless of a later sync; only a future rollout would fail to start if a required Secret were missing. `web-connector` is different again — it has no Secret at all; its credentials are baked into the image at **build time** via build arguments, not supplied at deploy time. `frontend` has no secrets to manage.

**Never commit real credentials to GitOpsRepo** — a chart's values file may document which keys a Secret needs as a placeholder, but that's documentation, not a real input path.

## Images

The hub, interface, investigation engine, and both connectors are custom-built images, each with its own `imageRegistry`/`image` value per chart; MongoDB, Qdrant, Ollama, and OVMS use public upstream images untouched by this monorepo's build pipeline. Building and publishing a new version of a custom image, and pointing the deployment configuration's values at that new tag, are two distinct, separately reviewed steps — publishing an image doesn't roll it out by itself.

## Manual install (no GitOps controller)

For a one-off cluster, or to debug a chart before wiring it into continuous delivery, the same chart set installs directly:

```
helm install mongo ./helm/charts/mongo -n xana --create-namespace
# provision qdrant's Secret first (see "Secrets" above), then:
helm install qdrant ./helm/charts/qdrant -n xana
# provision the hub's Secret first, then:
helm install backend ./helm/charts/backend -n xana
helm install frontend ./helm/charts/frontend -n xana
# provision the investigation engine's Secret first, same pattern, then:
helm install workflow-agent ./helm/charts/workflow-agent -n xana
# optional connectors:
# web-connector's credentials are build-time (see "Secrets" above) — no secret needed here:
helm install web-connector ./helm/charts/web-connector -n xana
# provision the Dynamics connector's Secret first, then:
helm install dynamics-connector ./helm/charts/dynamics-connector -n xana
# vision-captioning backend — pick one:
helm install ollama ./helm/charts/ollama -n xana
helm install ovms ./helm/charts/ovms -n xana
```

Run from inside a GitOpsRepo checkout — this is the same chart set Fleet installs, useful for a one-off cluster or debugging a chart before wiring it into Continuous Delivery. A bare `--set secrets.x=...` isn't a working shortcut for any of these charts — see "Secrets" above for the real per-chart Secret name/keys to provision instead.

## Where to go next

- What each of these charts actually runs: [The interface](../architecture/frontend.md) · [The hub](../architecture/backend.md) · [The investigation engine](../architecture/workflow-agent.md) · [Connectors](../architecture/connectors-web.md)
- Local, single-host deployment: [Running it locally](docker-deployment.md)
