# 1. Setup

> [Documentation](../README.md) → [Guides](README.md) → Setup
> Next: [2. First login](02-first-login.md)

XANA is a small set of independent services running together as one product. Getting a new deployment ready comes down to a short list of things someone needs to decide and provide up front, regardless of whether it's a quick local evaluation or a full production rollout — see [Deployment](../setup/docker-deployment.md) for both paths.

## What you'll be asked for

| What | Why |
|---|---|
| An AI model to use | XANA doesn't ship its own — you point it at an AI provider of your choice (see [4. Connecting data sources](04-connectors.md)). This is needed for workbench investigations and for AI-assisted setup steps. |
| An initial administrator | The very first account, created once, that signs in to do the rest of the setup below. |
| A database connection | Where all of XANA's durable data — people, projects, connectors, workbenches — actually lives. |

Everything else (retrieval tuning, optional document-understanding features, optional web search) ships with sensible defaults and only needs attention if you know you want to change it.

## Verifying it's up

Once a deployment is running, the interface, the hub, and the investigation engine each expose a simple health check, so confirming everything is reachable before you start using it is quick. If all three respond, continue to [2. First login](02-first-login.md).
