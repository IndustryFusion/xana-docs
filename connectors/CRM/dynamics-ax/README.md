# Dynamics CRM Adapter

A lightweight ASP.NET Core adapter that exposes Microsoft Dynamics CRM (on-premise) data through a REST API. This adapter authenticates via ADFS, queries the Dynamics OData API, and provides an OpenXANA-compliant connector interface for accessing CRM entities like accounts, cases, assets, and product/component cards.

## What It Does

This adapter serves as a bridge between OpenXANA and Microsoft Dynamics CRM installations secured with Active Directory Federation Services (ADFS). It:

- **Authenticates** against ADFS using forms-based authentication with username/password credentials
- **Caches** authentication cookies (FedAuth/MSISAuth) with a 30-minute TTL to minimize authentication overhead
- **Exposes** Dynamics CRM entities through a simplified REST API built on top of the Dynamics OData v8.2 endpoint
- **Implements** the OpenXANA connector contract v1.1.0, providing:
  - `/openxana/manifest` – Resource hierarchy and API structure
  - `/metadata` – OData service document listing all available entity sets
  - `/whoami` – Current authenticated user information
  - `/accounts`, `/incidents`, `/assets`, etc. – Entity-specific endpoints
- **Supports** hierarchical resource navigation (e.g., accounts → assets → componentcards)
- **Handles** pagination with OData `@odata.nextLink` continuation tokens

### Key Features

- Health check endpoint (`/healthz`) for Kubernetes liveness/readiness probes
- Global exception handler with appropriate HTTP status codes
- Cookie caching for improved performance
- Minimal footprint: .NET 8 ASP.NET Core

---

## Building and Running with Docker

### Prerequisites

- Docker installed on your machine
- Access to a Dynamics CRM instance secured with ADFS
- Valid CRM credentials (username and password)

### Build the Docker Image

From the `DynamicsAX` project directory:

```bash
docker build -t dynamics-adapter:latest .
```

The Dockerfile uses a multi-stage build:
1. **Build stage**: Uses `mcr.microsoft.com/dotnet/sdk:8.0` to restore dependencies and publish the application
2. **Runtime stage**: Uses `mcr.microsoft.com/dotnet/aspnet:8.0` with only the compiled binaries

### Run the Container Locally

```bash
docker run -d \
  --name dynamics-adapter \
  -p 8080:8080 \
  -e CRM_URL="https://your-crm-instance.domain.com" \
  -e CRM_USERNAME="domain\\username" \
  -e CRM_PASSWORD="your-password" \
  dynamics-adapter:latest
```

**Environment Variables:**

| Variable       | Description                                      | Example                          |
|----------------|--------------------------------------------------|----------------------------------|
| `CRM_URL`      | Base URL of your Dynamics CRM instance           | `https://your-crm.domain.com`    |
| `CRM_USERNAME` | Domain-qualified username for ADFS authentication| `domain\\username`              |
| `CRM_PASSWORD` | Password for the CRM user                        | `your-password-here`             |
| `CRM_ORG`      | (Optional) Organization name in the CRM URL      | Leave empty if not needed        |

### Verify the Container

```bash
# Health check
curl http://localhost:8080/healthz

# Get OpenXANA manifest
curl http://localhost:8080/openxana/manifest

# Verify authentication
curl http://localhost:8080/whoami
```

---

## OpenXANA Manifest Structure

The adapter exposes a **`GET /openxana/manifest`** endpoint that returns a JSON document describing all available resources, their API paths, parameters, field mappings, and hierarchical relationships. This manifest follows the OpenXANA Connector Contract v1.1.0.

### Manifest Overview

```bash
curl http://localhost:8080/openxana/manifest
```

The manifest defines:

| Field              | Description                                                  |
|--------------------|--------------------------------------------------------------|
| `connectorId`      | Unique identifier for this connector (`"dynamics-crm"`)      |
| `contractVersion`  | OpenXANA contract version implemented (`"1.1.0"`)            |
| `domain`           | Business domain category (`"support"`)                       |
| `resources`        | Collection of available resource types and their operations  |

### Resource Structure

Each resource in the manifest follows this structure:

#### Top-Level Properties

- **`type`**: Resource type identifier (e.g., `"account"`, `"case"`, `"asset"`)
- **`list`**: Configuration for listing/querying multiple resources
- **`detail`**: Configuration for retrieving a single resource by ID
- **`fields`**: Mapping between OpenXANA field names and Dynamics CRM field names
- **`children`**: Nested resources accessible under this parent resource
- **`rules`**: (Optional) Business rules for field values (e.g., status code labels)

#### List Operation

The `list` object defines how to retrieve a collection of resources:

```json
{
  "method": "GET",
  "path": "/accounts",
  "listPath": "value",
  "nextLinkPath": "@odata.nextLink",
  "parameters": {
    "pageSize": { "type": "integer", "default": 50 },
    "nextLink": { "type": "string" }
  }
}
```

- **`method`**: HTTP method (`"GET"`)
- **`path`**: API endpoint path
- **`listPath`**: JSON path to the array of results in the response (`"value"` for OData)
- **`nextLinkPath`**: JSON path to the pagination continuation token
- **`parameters`**: Available query parameters with their types and defaults

**Example**: List accounts with pagination

```bash
curl "http://localhost:8080/accounts?pageSize=100"
```

#### Detail Operation

The `detail` object defines how to retrieve a single resource:

```json
{
  "method": "GET",
  "path": "/accounts/{accountId}",
  "pathParams": {
    "accountId": "account.id"
  }
}
```

- **`pathParams`**: Maps path variables to resource field names

**Example**: Get account details

```bash
curl "http://localhost:8080/accounts/12345678-abcd-1234-abcd-123456789012"
```

#### Field Mappings

The `fields` object maps OpenXANA standard field names to Dynamics CRM internal field names:

```json
{
  "id": "accountid",
  "name": "name"
}
```

This allows OpenXANA to use consistent field names across different CRM systems while the adapter handles the translation to Dynamics-specific names.

#### Hierarchical Resources (Children)

Resources can have child resources accessible through nested paths. Children can be either **lists** (collections) or **single objects**:

**List children** (include `listPath`):
```json
{
  "assets": {
    "type": "asset",
    "path": "/accounts/{accountId}/assets",
    "listPath": "value"
  }
}
```

**Single object children** (no `listPath`):
```json
{
  "owner": {
    "type": "owner",
    "path": "/incidents/{caseId}/technicians",
    "description": "Incident owner (systemuser, team, or queue)"
  }
}
```

**Example**: List all incidents for an account

```bash
curl "http://localhost:8080/accounts/12345678-abcd-1234-abcd-123456789012/incidents"
```

**Example**: Get the owner of an incident

```bash
curl "http://localhost:8080/incidents/12345678-abcd-1234-abcd-123456789012/technicians"
# Returns: systemuser/team/queue object or {"owner": null}
```

#### Business Rules

Some resources define rules for interpreting field values:

```json
{
  "rules": {
    "activeStatus": {
      "field": "statuscode",
      "activeValues": [1, 2, 3, 4],
      "closedValues": [5, 6],
      "labels": {
        "1": "In Progress",
        "2": "On Hold",
        "3": "Waiting for Details",
        "4": "Researching",
        "5": "Problem Solved",
        "6": "Canceled"
      }
    }
  }
}
```

This tells clients which status codes represent active vs. closed cases and provides human-readable labels.

### Available Resources

The adapter exposes the following resource types:

| Resource          | Type            | List Path        | Child Resources                                                |
|-------------------|-----------------|------------------|----------------------------------------------------------------|
| **account**       | `account`       | `/accounts`      | assets, articlecards, productcards, componentcards, incidents  |
| **case**          | `case`          | `/incidents`     | activities, owner, notes                                       |
| **asset**         | `product`       | `/assets`        | componentcards                                                 |

#### Account Resource

**List parameters:**
- `pageSize` (integer, default: 50)
- `nextLink` (string) - OData continuation token

**Fields:**
- `id` → `accountid`
- `name` → `name`

**Child resources:**
- `assets` → `/accounts/{accountId}/assets`
- `articlecards` → `/accounts/{accountId}/articlecards`
- `productcards` → `/accounts/{accountId}/productcards`
- `componentcards` → `/accounts/{accountId}/componentcards`
- `incidents` → `/accounts/{accountId}/incidents`

#### Case Resource

**List parameters:**
- `pageSize` (integer, default: 50)
- `nextLink` (string) - OData continuation token
- `statuscode` (integer) - Filter by status
- `prioritycode` (integer) - Filter by priority
- `severitycode` (integer) - Filter by severity

**Fields:**
- `id` → `incidentid`
- `title` → `title`
- `status` → `statuscode`
- `priority` → `prioritycode`
- `accountId` → `_customerid_value`
- `accountName` → `customeridname`
- `primaryContactId` → `_primarycontactid_value`

**Status codes:**
- Active: 1 (In Progress), 2 (On Hold), 3 (Waiting for Details), 4 (Researching)
- Closed: 5 (Problem Solved), 6 (Canceled)

**Child resources:**
- `activities` → `/incidents/{caseId}/activities` (list)
- `owner` → `/incidents/{caseId}/technicians` (single object: systemuser, team, or queue)
- `notes` → `/incidents/{caseId}/notes` (list)

**Example**: List all active high-priority cases

```bash
curl "http://localhost:8080/incidents?statuscode=1&prioritycode=1&pageSize=25"
```

### Manifest Usage

The manifest enables OpenXANA clients to:

1. **Discover** available resources without hardcoding endpoints
2. **Navigate** hierarchical relationships (e.g., account → incidents → notes)
3. **Translate** field names between OpenXANA and Dynamics CRM
4. **Interpret** business rules (e.g., status code meanings)
5. **Paginate** large result sets using OData continuation tokens

---

## API Endpoints

### Utility Endpoints

- **`GET /healthz`** – Health check (returns `"healthy"`)
- **`GET /metadata`** – OData service document listing all entity sets
- **`GET /whoami`** – Current user's `systemuser` record
- **`GET /openxana/manifest`** – OpenXANA connector manifest (v1.1.0)

### Entity Endpoints (Examples)

- **`GET /accounts?pageSize=50`** – List accounts with pagination
- **`GET /accounts/{accountId}`** – Get account details
- **`GET /accounts/{accountId}/assets`** – List assets for an account
- **`GET /incidents?statuscode=1`** – List cases with filters
- **`GET /incidents/{caseId}`** – Get case details

Refer to the `/openxana/manifest` endpoint for the complete resource hierarchy and parameter schema.

---

## Kubernetes Deployment

The adapter includes Kubernetes manifests in the `k8s/` directory for production deployment:

- **`secret.yaml`** – Stores CRM credentials (CRM_URL, CRM_USERNAME, CRM_PASSWORD)
- **`deployment.yaml`** – Defines the adapter workload with health probes and resource limits
- **`service.yaml`** – Exposes the adapter as a ClusterIP service

**Quick deploy:**

```bash
# Create secret with credentials
kubectl create secret generic dynamics-adapter-secret \
  --from-literal=CRM_URL=https://your-crm.domain.com \
  --from-literal=CRM_USERNAME='domain\username' \
  --from-literal=CRM_PASSWORD='your-password'

# Deploy the adapter
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Verify
kubectl get pods -l app=dynamics-adapter
kubectl port-forward svc/dynamics-adapter 8080:80
curl http://localhost:8080/healthz
```

See the `k8s/` directory for detailed configuration options.

---

## Configuration

### appsettings.json

The application uses minimal configuration. Logging levels can be adjusted:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "DynamicsAdapter": "Debug"
    }
  }
}
```

### Environment Variables

All runtime configuration is handled via environment variables (see [Run the Container Locally](#run-the-container-locally)).

---

## Troubleshooting

### Authentication Failures

**Symptom**: `InvalidOperationException: ADFS did not return a WS-Federation token form`

**Possible causes**:
- Incorrect `CRM_USERNAME` or `CRM_PASSWORD`
- Username not in `domain\username` format
- ADFS configured for Windows Integrated Auth only (the adapter forces forms auth)

**Solution**: Verify credentials and ensure ADFS supports forms-based authentication.

### Connection Errors

**Symptom**: `HttpRequestException` or 502 Bad Gateway

**Possible causes**:
- `CRM_URL` unreachable from the container/pod
- Network policies blocking egress traffic
- CRM instance down or misconfigured

**Solution**: Test connectivity with `curl` from within the container/pod.

### Cookie Expiration

Cookies are cached for 30 minutes. If you see repeated authentication calls, check the `_cookiesFetchedAt` logic in [DynamicsService.cs](Services/DynamicsService.cs).

---

## License

Copyright (c) 2026 IndustryFusion Europe UG

Licensed under the Apache License, Version 2.0. See [LICENSE](../../../LICENSE) for details.
