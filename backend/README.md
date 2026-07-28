# XANA Business Backend

NestJS API for support workbenches, CRM connectors, knowledge browsing, and resolution workflows.

## MongoDB (required)

All durable backend state is stored in MongoDB. Set these environment variables in `.env`:

```env
MONGODB_URI=mongodb://localhost:27017/xana-business
# Optional when the database name is not in the URI:
# MONGODB_DB_NAME=xana-business
```

### Collections

| Collection | Purpose |
|------------|---------|
| `workbenches` | Support workbench sessions (steps, analysis, messages) |
| `connectors` | Registered OpenXANA connector configs |
| `connector_response_cache` | Cached connector HTTP responses (7-day TTL by default) |
| `resolution_states` | Case copilot analyses, feedback, drafts |
| `case_support_state` | Per-case product mappings and attachment metadata |
| `connector_mappings` | AI-generated CRM endpoint/field mappings |

Short-lived in-memory caches (knowledge folder TTL, in-flight request dedup) are **not** stored in MongoDB.

### Migrate legacy JSON files

If you have existing data under `.data/`, migrate once:

**Option A — on startup**

```env
MONGODB_MIGRATE_FROM_FILES=true
```

**Option B — CLI script**

```bash
MONGODB_URI=mongodb://localhost:27017/xana-business npx ts-node scripts/migrate-json-to-mongo.ts
```

Migration skips any collection that already contains documents.

### Connector cache TTL

```env
# Default: 604800 (7 days). Set to 0 to disable TTL index.
CONNECTOR_CACHE_TTL_SECONDS=604800
```

## Run

```bash
npm install
npm run start:dev
```

## Tests

```bash
npm test
```

Uses `mongodb-memory-server` for repository integration tests (no external MongoDB required for tests).
