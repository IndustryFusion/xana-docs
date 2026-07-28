# OpenXANA Web Connector Service

Configurable REST connector for authorized websites, wikis, and document portals. It authenticates with a configured site, keeps a cookie or token session, parses HTML folder pages into structured JSON, and streams validated document downloads through the backend.

This is not a general internet crawler. Every remote request is constrained by connector config, allowed hosts, URL validation, SSRF checks, timeouts, crawl limits, and file-size limits.

## Run Locally

```bash
npm install
npm run build
PORT=8080 WEB_CONNECTOR_CONFIG_DIR=./config npm run start:prod
```

Health check:

```bash
curl http://localhost:8080/healthz
```

Discovery manifest:

```bash
curl http://localhost:8080/openxana/manifest
```

## Docker

```bash
docker build -t openxana-web-connector:latest .
docker run --rm -p 8080:8080 \
  -e WEB_CONNECTOR_CONFIG_DIR=/data/connectors \
  -e WIKI_USERNAME='admin@example.com' \
  -e WIKI_PASSWORD='change-me' \
  -v web-connector-data:/data/connectors \
  openxana-web-connector:latest
```

Place one or more connector JSON files in the mounted config directory before starting the service. The filename becomes the connector id. For example, `/data/connectors/company-wiki.json` is exposed as connector id `company-wiki`.

## Endpoints

`GET /openxana/manifest` is the entry point for XANA Business or another AI/UI client. It describes the available resources, paths, parameters, response shapes, and the UI navigation flow.

UI/client browsing endpoints:

- `GET /web-connectors` lists configured connector summaries so a UI can choose a portal.
- `GET /web-connectors/:id/folders` fetches the configured document root URL and returns its folders, pages, files, and warnings.
- `GET /web-connectors/:id/folders/:folderId` lists direct child folders/files/pages for a folder selected from a previous response.
- `GET /web-connectors/:id/folders/:folderId/tree?depth=2` returns a bounded recursive tree from a selected folder.
- `GET /web-connectors/:id/documents/:documentId/metadata` returns display metadata for a selected file/PDF.
- `GET /web-connectors/:id/documents/:documentId/download` streams a selected file/PDF through the backend.

Normal UI flow:

1. Call `GET /web-connectors` and select a connector id, such as `company-wiki`.
2. Call `GET /web-connectors/company-wiki/folders`.
3. Render the returned `folders` and `files` arrays.
4. When the user selects a folder, call its returned `childrenEndpoint`.
5. When the user selects a document/PDF, call its returned `metadataEndpoint` or `downloadEndpoint`.

Example folder item:

```json
{
  "id": "aHR0cHM6Ly93aWtpLmV4YW1wbGUuY29tL2RvY3VtZW50cy9tYW51YWxz",
  "name": "Manuals",
  "type": "folder",
  "childrenEndpoint": "/web-connectors/company-wiki/folders/aHR0cHM6Ly93aWtpLmV4YW1wbGUuY29tL2RvY3VtZW50cy9tYW51YWxz",
  "metadata": {}
}
```

Example file item:

```json
{
  "id": "aHR0cHM6Ly93aWtpLmV4YW1wbGUuY29tL2Rvd25sb2FkL21hbnVhbC5wZGY",
  "name": "Manual.pdf",
  "type": "pdf",
  "extension": "pdf",
  "contentType": "application/pdf",
  "metadataEndpoint": "/web-connectors/company-wiki/documents/aHR0cHM6Ly93aWtpLmV4YW1wbGUuY29tL2Rvd25sb2FkL21hbnVhbC5wZGY/metadata",
  "downloadEndpoint": "/web-connectors/company-wiki/documents/aHR0cHM6Ly93aWtpLmV4YW1wbGUuY29tL2Rvd25sb2FkL21hbnVhbC5wZGY/download",
  "metadata": {}
}
```

Admin/configuration endpoints:

- `GET /web-connectors/:id` returns a sanitized config summary.
- `POST /web-connectors/:id/test-auth` tests remote authentication.
- `POST /web-connectors/:id/refresh-session` forces a new remote session.

The UI should follow the returned endpoint fields. It does not need to copy remote URLs into query parameters. The backend still validates every decoded folder/document id against the connector's allowlist and SSRF rules before fetching.

Connector configs are deployment inputs, not runtime API resources. Mount them through Docker volumes, Kubernetes ConfigMaps, or another read-only file source and restart/redeploy the service when configuration changes.

## Example Config

Credentials should use secret placeholders or `secretRef` values. `{{WIKI_USERNAME}}` resolves from the environment variable `WIKI_USERNAME`.

```json
{
  "name": "Example Wiki",
  "baseUrl": "https://wiki.example.com",
  "allowedHosts": ["wiki.example.com"],
  "auth": {
    "type": "form",
    "loginUrl": "https://wiki.example.com/login",
    "method": "POST",
    "contentType": "application/x-www-form-urlencoded",
    "credentialPlacement": "body",
    "fields": {
      "username": "{{WIKI_USERNAME}}",
      "password": "{{WIKI_PASSWORD}}"
    },
    "successDetection": {
      "type": "notContainsHtml",
      "selectorOrText": "login"
    },
    "session": {
      "type": "cookie",
      "expiresInSeconds": 3600
    }
  },
  "roots": [
    {
      "name": "Root",
      "url": "https://wiki.example.com/documents"
    }
  ],
  "parsing": {
    "itemSelector": "a[href]",
    "nameSelector": null,
    "hrefAttribute": "href",
    "folderRules": [{ "type": "urlContains", "value": "/folder/" }],
    "fileRules": [
      { "type": "extension", "values": [".pdf", ".docx", ".xlsx", ".pptx", ".zip"] },
      { "type": "urlContains", "value": "/download/" }
    ],
    "ignoreRules": [
      { "type": "urlContains", "value": "logout" },
      { "type": "urlStartsWith", "value": "mailto:" },
      { "type": "urlStartsWith", "value": "javascript:" }
    ],
    "metadataSelectors": {
      "title": "title",
      "lastModified": ".modified-date",
      "size": ".file-size"
    },
    "pagination": {
      "enabled": false,
      "nextSelector": "a.next"
    }
  },
  "limits": {
    "maxDepth": 3,
    "maxPages": 100,
    "maxFileSizeMb": 100,
    "requestTimeoutMs": 30000,
    "maxConcurrentRequests": 4
  }
}
```

## Security Defaults

- Only configured `allowedHosts` can be fetched.
- Localhost, private IP ranges, link-local addresses, multicast/reserved ranges, and metadata service IPs are blocked.
- `http://` URLs are disabled unless `WEB_CONNECTOR_ALLOW_HTTP=true`.
- Private-network access requires both connector `allowPrivateNetwork=true` and `WEB_CONNECTOR_ALLOW_PRIVATE_NETWORK=true`.
- Plaintext credentials are rejected by default; use env placeholders or secret refs.
- Cookies and tokens are kept in memory and redacted from logs/responses.
- Tree crawling enforces max depth, max pages, timeout, and visited URL protection.
- Downloads enforce max file size and stream without buffering the full file.

## Production Notes

The default repository stores connector configs as JSON files under `WEB_CONNECTOR_CONFIG_DIR`. Use a persistent Docker volume for local deployments. For multi-replica production deployments, replace the repository and session store with platform services such as Postgres/MongoDB for configs, Redis for sessions, and Kubernetes/Vault/cloud secret manager for credentials.
