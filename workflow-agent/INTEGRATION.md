# Workbench ↔ Workflow Agent Integration

## Architecture

```
Frontend (Next.js)
  POST /support/:workspaceId/workbenches/:id/analyze
        ↓
NestJS WorkbenchesService.analyze()
        ↓
WorkflowAgentClient → POST http://workflow-agent:8000/v1/workbench/analyze
        ↓
LangGraph workflow (enrich_context → plan_and_generate)
        ↓
Tools call XANA APIs (CRM + knowledge)
        ↓
Response mapped to AnalysisRun + TroubleshootingStep + copilot message
        ↓
Full SupportWorkbench returned to UI
```

**No frontend changes are required.** The workbench detail page already calls `api.workbenches.analyze()`.

## Enable in NestJS backend

Add to `backend/.env`:

```env
WORKFLOW_AGENT_ENABLED=true
WORKFLOW_AGENT_URL=http://localhost:8000
WORKFLOW_AGENT_TIMEOUT_MS=120000
WORKFLOW_AGENT_LANGUAGE=en
```

| Variable | Purpose |
|----------|---------|
| `WORKFLOW_AGENT_ENABLED` | `true` to call Python agent; `false` uses rule-based fallback |
| `WORKFLOW_AGENT_URL` | Base URL of workflow-agent service |
| `WORKFLOW_AGENT_TIMEOUT_MS` | HTTP timeout for analyze (LLM can be slow) |
| `WORKFLOW_AGENT_LANGUAGE` | `en` or `de` for generated step text |

Restart the NestJS backend after changing env vars.

## Enable workflow-agent

Add to `workflow-agent/.env` (copy from `backend/.env` for LLM vars):

```env
XANA_API_BASE_URL=http://localhost:4000
OPENAI_COMPATIBLE_BASE_URL=...
OPENAI_COMPATIBLE_API_KEY=...
OPENAI_COMPATIBLE_MODEL=...
AGENT_PORT=8000
```

## Docker + local backend

1. Start NestJS backend on port 4000
2. Start workflow-agent:

```bash
cd workflow-agent
docker compose up --build
```

3. Set backend env:

```env
WORKFLOW_AGENT_ENABLED=true
WORKFLOW_AGENT_URL=http://localhost:8000
```

## Docker Compose (all services)

If you later add backend to compose, use service names:

```env
WORKFLOW_AGENT_URL=http://workflow-agent:8000
XANA_API_BASE_URL=http://backend:4000
```

## User flow (unchanged in UI)

1. Create workbench with CRM case + knowledge scope + optional incidents
2. Open workbench detail
3. Click **Get repair steps**
4. Backend calls workflow-agent
5. UI shows analysis, evidence, and step cards

## Fallback behavior

If `WORKFLOW_AGENT_ENABLED` is not `true`, or the agent is unreachable, NestJS falls back to the existing rule-based `analyze()` logic. The workbench still works without the Python service.

## Files changed in xana-business

| File | Change |
|------|--------|
| `workflow-agent/` | New Python LangGraph service |
| `backend/src/workbenches/workflow-agent.client.ts` | HTTP client to agent |
| `backend/src/workbenches/workbenches.service.ts` | `analyze()` calls agent first |
| `backend/src/workbenches/workbenches.module.ts` | Registers `HttpModule` + client |

## Phase 2 (optional)

Wire `POST /v1/workbench/continue` when technician uses **Ask for help** or marks a step failed:

- Add `WorkflowAgentClient.continueWorkbench()`
- Call from `addMessage()` or new `POST .../continue` endpoint
- Append copilot reply and optional extra steps to workbench
