# Running it in production

> [Documentation](../README.md) → Running it in production
> See also: [Running it locally](docker-deployment.md) (local/single-host) · [root README](../../README.md)

Beyond a local look, XANA runs on Kubernetes as one independently deployed and scaled workload per service. Application code and deployed cluster state are versioned and reviewed independently, on purpose: a production rollout is reviewed and reversible the same way a code change is, rather than something anyone applies to a cluster by hand.

## How it's deployed

XANA's cluster configuration is managed through GitOps: changes to what's running are made by committing a change to a dedicated deployment configuration, and a continuous-delivery controller reconciles the cluster to match it automatically. This means:

- No one runs deployment commands against production directly — a change goes through review like any other change.
- Rolling back is as simple as reverting the configuration change that caused a problem.
- Every service — the interface, the hub, the investigation engine, the database and retrieval index, and each connector — is deployed and can be upgraded on its own, without redeploying the rest.

## Configuration and secrets

Day-to-day configuration (which AI model to default to, feature toggles, resource sizing) is managed the same GitOps way as everything else. Credentials and other sensitive values are handled deliberately outside of that same reviewed configuration flow, so they're never sitting in plain text in a reviewable history — they're provisioned directly into the cluster by whoever operates it.

## Images

Each service's deployable artifact is built from this repository and published to a container registry, versioned per release; the production configuration then points at a specific version of each. Publishing a new version and pointing production at it are two distinct, separately reviewed steps.

## Where to go next

- What each of these services actually runs: [The interface](../architecture/frontend.md) · [The hub](../architecture/backend.md) · [The investigation engine](../architecture/workflow-agent.md) · [Connectors](../architecture/connectors-web.md)
- Local, single-machine deployment: [Running it locally](docker-deployment.md)
