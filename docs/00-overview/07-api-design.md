╔═════════════════════════════════════════════════════════════════════════════════════════════════╗
║ Document ID: DOC-07                                                                            ║
║ Title: API Design — OpenAPI 3.1, JWT, Multi-Tenancy, BFF Contracts                             ║
║ Version: 1.0.0                                                                                 ║
║ Status: ✓ Approved                                                                             ║
║ Classification: Internal — Confidential                                                        ║
║ Authors: NexusAsset Cloud API Team                                                             ║
║ Reviewers: Frontend, Backend, Security, Platform, Developer Experience                         ║
║ Approved By: CTO / VP Engineering / API Governance Board                                       ║
║ Date: 2026-11-09                                                                               ║
║ Supersedes: —                                                                                  ║
║ Related: DOC-00, DOC-01, DOC-02, DOC-03, DOC-04, DOC-05, DOC-06, DOC-09, DOC-13              ║
║ Traceability: API/CTX-### ← REQ-FN-###/UC-###, API/CTX-■ → OpenAPI Spec, ADR-### → API Decisions ║
╚════════════════════════════════════════════════════════════════════════════════════════════════╝

# ═══════════════════════════════════════════════════════════════
# NexusAsset Cloud — API Design (DOC-07)
# ═══════════════════════════════════════════════════════════════

> **Executive Summary** — This document defines the API design standards and contracts for NexusAsset Cloud. The API follows **OpenAPI 3.1** specification with **JSON Schema 2020-12** for validation. Authentication uses **JWT (RS256)** with short-lived access tokens (15 min) and rotating refresh tokens (30 days). Multi-tenancy is enforced via **tenant context resolution** (subdomain, header, JWT claim). The **BFF (Backend-for-Frontend)** pattern provides optimized endpoints for the Next.js frontend. All APIs are versioned via URL path (`/api/v1/`), with semantic versioning for breaking changes. The canonical source of truth is `api/contracts/openapi.yaml` in the monorepo.

---

## Table of Contents

1. [API Design Principles](#1-api-design-principles)
2. [Authentication & Authorization](#2-authentication--authorization)
3. [Multi-Tenancy Strategy](#3-multi-tenancy-strategy)
4. [API Versioning & Lifecycle](#4-api-versioning--lifecycle)
5. [Request/Response Standards](#5-requestresponse-standards)
6. [Error Handling](#6-error-handling)
7. [Pagination, Filtering & Sorting](#7-pagination-filtering--sorting)
8. [Rate Limiting & Quotas](#8-rate-limiting--quotas)
9. [Webhooks & Event Notifications](#9-webhooks--event-notifications)
10. [BFF (Backend-for-Frontend) Contracts](#10-bff-backend-for-frontend-contracts)
11. [OpenAPI Specification Structure](#11-openapi-specification-structure)
12. [Core API Endpoints](#12-core-api-endpoints)
13. [Developer Portal & SDKs](#13-developer-portal--sdks)
14. [API Governance & Compliance](#14-api-governance--compliance)
15. [Diagrams](#15-diagrams)
16. [Tables](#16-tables)

---

## 1. API Design Principles

### 1.1 Design Principles

| Principle ID | Principle | Description |
|--------------|-----------|-------------|
| **API-PRIN-001** | **Contract-First** | OpenAPI spec is source of truth; code generated from spec |
| **API-PRIN-002** | **Resource-Oriented** | RESTful resources with standard HTTP verbs; RPC for actions |
| **API-PRIN-003** | **Tenant-First** | Every request scoped to tenant; cross-tenant access forbidden |
| **API-PRIN-004** | **Explicit Versioning** | Version in URL path (`/api/v1/`); headers for minor versions |
| **API-PRIN-005** | **Idempotency** | All mutating endpoints support idempotency keys |
| **API-PRIN-006** | **Consistent Errors** | RFC 9457 Problem Details format for all errors |
| **API-PRIN-007** | **Cursor Pagination** | No offset pagination; opaque cursors for consistency |
| **API-PRIN-008** | **Schema Validation** | Request/response validated against OpenAPI schemas |
| **API-PRIN-009** | **Observability** | Correlation IDs, structured logs, metrics on every request |
| **API-PRIN-010** | **Backward Compatibility** | Non-breaking changes only within major version |

### 1.2 API Style Guide

| Aspect | Standard |
|--------|----------|
| **Base URL** | `https://api.nexusasset.cloud/api/v1` |
| **Protocol** | HTTPS only (TLS 1.3) |
| **Content-Type** | `application/json` (request/response) |
| **Date Format** | RFC 3339 (e.g., `2026-11-09T10:30:45.123Z`) |
| **UUID Format** | UUID v7 (timestamp-ordered) |
| **Naming** | snake_case for fields, kebab-case for URLs |
| **Enums** | UPPER_SNAKE_CASE (e.g., `ASSET_STATE_ACTIVE`) |
| **Boolean Fields** | `is_`, `has_`, `can_` prefix |
| **Timestamps** | `_at` suffix (created_at, updated_at, deleted_at) |
| **IDs** | `_id` suffix (asset_id, tenant_id, user_id) |

---

## 2. Authentication & Authorization

### 2.1 JWT Token Structure

**Access Token (JWT, RS256, 15 min TTL)**:
```json
{
  "header": { "alg": "RS256", "typ": "JWT", "kid": "nxa-key-2026-01" },
  "payload": {
    "iss": "https://auth.nexusasset.cloud",
    "sub": "usr_abc123def456",
    "tid": "tn_xyz789ghi012",
    "iat": 1704799845,
    "exp": 1704800745,
    "jti": "0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6",
    "roles": ["asset_manager", "tenant_admin"],
    "permissions": ["assets:read", "assets:write", "assets:delete", "tenants:read"],
    "scopes": ["api:read", "api:write"],
    "mfa_verified": true,
    "session_id": "ses_0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6"
  }
}
```

**Refresh Token (Opaque, 30 days, rotating)**:
- Stored as bcrypt hash in database
- Rotated on each use (new refresh token issued, old revoked)
- Revoked on logout, password change, security event

### 2.2 Authentication Flows

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              AUTHENTICATION FLOWS                                                   │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  1. USERNAME/PASSWORD + MFA                                                                        │
│  ┌─────────┐     POST /api/v1/auth/login     ┌─────────────────────────────────────────────────┐  │
│  │ Client  │ ──────────────────────────────▶ │ nxa-backend                                     │  │
│  │         │ ◀────────────────────────────── │  • Verify Argon2id hash                         │  │
│  │         │   200 OK + access + refresh     │  • Check MFA requirement                        │  │
│  └─────────┘                                 │  • If MFA: return 401 with mfa_required=true  │  │
│                                             │  • Issue JWT pair                                 │  │
│                                             └─────────────────────────────────────────────────┘  │
│                                             │  If MFA required:                                │  │
│                                             │  POST /api/v1/auth/mfa/verify                    │  │
│                                             │  { "method": "totp", "code": "123456" }          │  │
│                                             │  → 200 OK + JWT pair                             │  │
│                                                                                                     │
│  2. SSO (SAML/OIDC)                                                                               │
│  ┌─────────┐     GET /api/v1/auth/sso/saml    ┌────────────────────────────────────────────┐  │
│  │ Client  │ ────────────────────────────────▶ │ nxa-backend (SP)                         │  │
│  │         │ ◀──────────────────────────────── │  • Redirect to IdP                        │  │
│  │         │   302 Redirect                   │  • IdP → POST /api/v1/auth/sso/acs        │  │
│  └─────────┘                                 │  • Validate assertion, JIT provision      │  │
│                                              │  • Issue JWT pair → redirect to app        │  │
│                                              └────────────────────────────────────────────┘  │
│                                                                                                     │
│  3. API KEY                                                                                       │
│  ┌─────────┐     Header: Authorization: Bearer nxa_ak_... ┌────────────────────────────────┐  │
│  │ Client  │ ────────────────────────────────────────────▶ │ nxa-backend                   │  │
│  │         │ ◀──────────────────────────────────────────── │  • Validate bcrypt hash       │  │
│  │         │   200 OK (scoped to key permissions)         │  • Inject user/tenant context │  │
│  └─────────┘                                              └────────────────────────────────┘  │
│                                                                                                     │
│  4. TOKEN REFRESH                                                                                 │
│  POST /api/v1/auth/refresh                                                                        │
│  { "refresh_token": "rt_..." }                                                                    │
│  → 200 OK { "access_token": "...", "refresh_token": "rt_new...", "expires_in": 900 }           │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Authorization Model

| Level | Mechanism | Enforcement Point |
|-------|-----------|-------------------|
| **Transport** | mTLS (service-to-service) | Cilium/Traefik |
| **Authentication** | JWT validation (signature, exp, iss, aud) | Traefik ForwardAuth → nxa-core |
| **Tenant Context** | `tid` claim / `X-Tenant-ID` header / subdomain | Middleware → `SET LOCAL app.tenant_id` |
| **RBAC** | Roles → Permissions → Endpoint requirements | DRF Permission classes |
| **ABAC** | OPA policies (resource tags, time, location, risk) | AuthorizationService in use cases |
| **Resource Ownership** | Asset owner/custodian checks | Domain invariants in aggregates |

---

## 3. Multi-Tenancy Strategy

### 3.1 Tenant Context Resolution

```http
# Priority order (highest wins):
# 1. Subdomain: GET https://{tenant}.app.nexusasset.cloud/api/v1/assets
# 2. Header: GET /api/v1/assets  Header: X-Tenant-ID: tn_abc123
# 3. JWT Claim: Authorization: Bearer <jwt>  (tid claim)
# 4. Default: Platform admin only (no tenant context)
```

### 3.2 Tenant Context Headers

| Header | Purpose | Required |
|--------|---------|----------|
| `X-Tenant-ID` | Explicit tenant ID (UUID) | For API keys, service-to-service |
| `X-Request-ID` | Correlation ID (UUID v7) | Generated by Traefik if missing |
| `X-Correlation-ID` | Upstream correlation ID | Passed through for tracing |

### 3.3 Cross-Tenant Access Prevention

- **Database**: RLS policies enforce `tenant_id = current_setting('app.tenant_id')`
- **API**: Middleware validates `tid` claim matches `X-Tenant-ID` / subdomain
- **Queries**: ORM automatically scopes all queries; raw SQL blocked by linter
- **Admin API**: Platform admins use separate `/api/v1/platform/*` endpoints with elevated permissions

---

## 4. API Versioning & Lifecycle

### 4.1 Versioning Strategy

```
URL Path:     /api/v1/assets          (Major version in path)
Header:       Accept: application/vnd.nexusasset.v1+json
              Accept-Version: v1.2     (Minor version for backward-compatible additions)
```

### 4.2 Version Lifecycle

| Stage | Major Version | Minor Version | Support |
|-------|---------------|---------------|---------|
| **Alpha** | v0 (internal only) | v0.1, v0.2 | No SLA |
| **Beta** | v1 (preview) | v1.1, v1.2 | Best effort |
| **GA** | v1, v2, ... | v1.1, v1.2 | Full SLA |
| **Deprecated** | v1 (marked) | v1.x | 12 months notice |
| **Sunset** | v1 (removed) | — | 410 Gone |

### 4.3 Deprecation Headers

```http
# Response headers for deprecated endpoints
Sunset: Sat, 01 Jan 2027 00:00:00 GMT
Deprecation: true
Link: <https://api.nexusasset.cloud/api/v2/assets>; rel="successor-version"
```

---

## 5. Request/Response Standards

### 5.1 Standard Request Headers

| Header | Description | Example |
|--------|-------------|---------|
| `Authorization` | Bearer token (JWT or API key) | `Bearer eyJhbGciOiJSUzI1NiIs...` |
| `X-Tenant-ID` | Tenant context (if not in JWT/subdomain) | `tn_abc123def456` |
| `X-Request-ID` | Client-generated correlation ID | `0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6` |
| `Idempotency-Key` | Idempotency key for mutations | `idem_0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6` |
| `Accept` | Response format/version | `application/vnd.nexusasset.v1+json` |
| `Content-Type` | Request body format | `application/json` |
| `If-Match` | Optimistic locking (ETag) | `W/"0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6"` |

### 5.2 Standard Response Headers

| Header | Description | Example |
|--------|-------------|---------|
| `X-Request-ID` | Echoed correlation ID | `0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6` |
| `X-RateLimit-Limit` | Request limit in window | `1000` |
| `X-RateLimit-Remaining` | Remaining requests | `999` |
| `X-RateLimit-Reset` | Unix timestamp of reset | `1704800400` |
| `ETag` | Resource version for caching | `W/"0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6"` |
| `Cache-Control` | Caching directives | `private, max-age=60` |

### 5.3 Success Response Envelope

```json
{
  "data": { ... },           // Single resource or array
  "meta": {                  // Optional metadata
    "request_id": "0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6",
    "timestamp": "2026-11-09T10:30:45.123Z",
    "version": "v1.2"
  },
  "links": {                 // HATEOAS links
    "self": "/api/v1/assets/ast_123",
    "next": "/api/v1/assets?cursor=eyJpZCI6ImFzdF8xMjMifQ",
    "prev": null
  }
}
```

### 5.4 Collection Response

```json
{
  "data": [
    { "id": "ast_001", "name": "Server 01", ... },
    { "id": "ast_002", "name": "Laptop 02", ... }
  ],
  "meta": {
    "request_id": "0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6",
    "timestamp": "2026-11-09T10:30:45.123Z",
    "total_count": 15420,
    "has_more": true
  },
  "links": {
    "self": "/api/v1/assets?limit=100",
    "next": "/api/v1/assets?cursor=eyJpZCI6ImFzdF8wMDIifQ&limit=100",
    "prev": null
  }
}
```

---

## 6. Error Handling

### 6.1 RFC 9457 Problem Details Format

```json
{
  "type": "https://api.nexusasset.cloud/errors/validation_error",
  "title": "Validation Failed",
  "status": 422,
  "detail": "Request body failed validation",
  "instance": "/api/v1/assets",
  "request_id": "0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6",
  "errors": [
    {
      "field": "serial_number",
      "code": "INVALID_FORMAT",
      "message": "Serial number must be alphanumeric, max 100 characters",
      "rejected_value": "INVALID@SERIAL!"
    },
    {
      "field": "asset_type",
      "code": "INVALID_ENUM",
      "message": "Invalid asset type. Must be one of: SERVER, LAPTOP, NETWORK_DEVICE, ...",
      "rejected_value": "INVALID_TYPE"
    }
  ]
}
```

### 6.2 Standard Error Types

| HTTP Status | Error Type | Type URI | When |
|-------------|------------|----------|------|
| 400 | Bad Request | `/errors/bad_request` | Malformed JSON, missing required fields |
| 401 | Unauthorized | `/errors/unauthorized` | Invalid/expired token, missing auth |
| 403 | Forbidden | `/errors/forbidden` | Insufficient permissions, tenant mismatch |
| 404 | Not Found | `/errors/not_found` | Resource doesn't exist or not in tenant |
| 409 | Conflict | `/errors/conflict` | Duplicate resource, version mismatch (ETag) |
| 422 | Unprocessable Entity | `/errors/validation_error` | Semantic validation failed |
| 429 | Too Many Requests | `/errors/rate_limited` | Rate limit exceeded |
| 500 | Internal Server Error | `/errors/internal_error` | Unexpected server error |
| 503 | Service Unavailable | `/errors/service_unavailable` | Dependency down, maintenance mode |

### 6.3 Error Codes (Machine-Readable)

| Code | HTTP | Description |
|------|------|-------------|
| `AUTH_TOKEN_EXPIRED` | 401 | Access token expired |
| `AUTH_TOKEN_INVALID` | 401 | Token signature invalid or malformed |
| `AUTH_MFA_REQUIRED` | 401 | MFA challenge required |
| `AUTH_MFA_INVALID` | 401 | MFA code invalid/expired |
| `TENANT_NOT_FOUND` | 404 | Tenant ID not found |
| `TENANT_SUSPENDED` | 403 | Tenant is suspended |
| `QUOTA_EXCEEDED` | 429 | Tenant quota exceeded |
| `RESOURCE_CONFLICT` | 409 | Duplicate unique field |
| `VERSION_CONFLICT` | 409 | Optimistic lock failure (ETag mismatch) |
| `IDEMPOTENCY_CONFLICT` | 409 | Idempotency key reused with different payload |
| `VALIDATION_FAILED` | 422 | Request validation failed |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

---

## 7. Pagination, Filtering & Sorting

### 7.1 Cursor-Based Pagination

```http
# First request
GET /api/v1/assets?limit=50

# Response includes next cursor
{
  "data": [...],
  "links": {
    "next": "/api/v1/assets?cursor=eyJpZCI6ImFzdF8wNTAiLCJzZWFyY2hfYWZ0ZXIiOiIyMDI2LTExLTA5VDEwOjMwOjQ1WiJ9&limit=50"
  }
}

# Subsequent request
GET /api/v1/assets?cursor=eyJpZCI6ImFzdF8wNTAiLCJzZWFyY2hfYWZ0ZXIiOiIyMDI2LTExLTA5VDEwOjMwOjQ1WiJ9&limit=50
```

**Cursor Format** (Base64 encoded JSON):
```json
{
  "id": "ast_050",
  "search_after": "2026-11-09T10:30:45Z"
}
```

### 7.2 Filtering (JMESPath-inspired)

```http
# Exact match
GET /api/v1/assets?filter[asset_type]=SERVER

# Multiple values (OR)
GET /api/v1/assets?filter[asset_type]=SERVER,LAPTOP

# Comparison operators
GET /api/v1/assets?filter[purchase_date][gte]=2024-01-01
GET /api/v1/assets?filter[purchase_cost][lte]=50000

# Nested fields
GET /api/v1/assets?filter[location.datacenter]=DC-EU-01

# Full-text search
GET /api/v1/assets?search=dell%20r750

# Saved query
GET /api/v1/assets?saved_query=high_value_servers
```

### 7.3 Sorting

```http
# Single field
GET /api/v1/assets?sort=created_at

# Descending
GET /api/v1/assets?sort=-created_at

# Multiple fields
GET /api/v1/assets?sort=-state,created_at,name
```

---

## 8. Rate Limiting & Quotas

### 8.1 Rate Limit Tiers

| Tier | Requests/Minute | Burst | Scope |
|------|-----------------|-------|-------|
| **Free** | 60 | 120 | Per tenant |
| **Pro** | 300 | 600 | Per tenant |
| **Enterprise** | 1000 | 2000 | Per tenant |
| **Platform Admin** | 10000 | 20000 | Per user |

### 8.2 Quota Headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1704800400
X-RateLimit-Window: 60
```

### 8.3 Quota Enforcement

- **Per-tenant**: Enforced at Traefik (Redis-backed token bucket)
- **Per-endpoint**: Configurable overrides (e.g., `/auth/login` stricter)
- **Burst handling**: 2x limit for short bursts
- **Exceeded**: 429 with `Retry-After` header

---

## 9. Webhooks & Event Notifications

### 9.1 Webhook Registration

```http
POST /api/v1/webhooks
{
  "url": "https://customer.example.com/webhooks/nexusasset",
  "events": ["asset.created", "asset.updated", "asset.state_changed", "compliance.finding_created"],
  "secret": "whsec_abc123...",  // Used for HMAC-SHA256 signature
  "retry_policy": {
    "max_attempts": 5,
    "backoff": "exponential",
    "initial_interval_seconds": 10,
    "max_interval_seconds": 3600
  },
  "headers": {
    "X-Custom-Header": "value"
  }
}
```

### 9.2 Webhook Payload

```json
{
  "id": "evt_0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6",
  "type": "asset.state_changed",
  "timestamp": "2026-11-09T10:30:45.123Z",
  "tenant_id": "tn_abc123",
  "data": {
    "asset_id": "ast_123456",
    "from_state": "ACTIVE",
    "to_state": "MAINTENANCE",
    "reason": "Scheduled preventive maintenance",
    "changed_by": "usr_xyz789"
  },
  "signature": "sha256=abc123..."
}
```

### 9.3 Signature Verification

```python
# Client verification
import hmac, hashlib
expected = hmac.new(
    secret.encode(),
    payload_bytes,
    hashlib.sha256
).hexdigest()
# Compare with header: X-NexusAsset-Signature: sha256=abc123...
```

---

## 10. BFF (Backend-for-Frontend) Contracts

### 10.1 BFF Purpose

The BFF layer (`nxa-frontend` Next.js API routes) provides:
- **Aggregated responses** (single request → multiple backend calls)
- **Frontend-optimized shapes** (exact fields needed by UI components)
- **Permission-aware filtering** (hide fields user can't access)
- **Caching** (React Query / SWR integration)

### 10.2 BFF Endpoint Examples

| BFF Endpoint | Backend Calls | Purpose |
|--------------|---------------|---------|
| `GET /bff/assets/dashboard` | `/assets?state=ACTIVE&limit=5`, `/assets/count`, `/compliance/score` | Dashboard overview |
| `GET /bff/assets/{id}/detail` | `/assets/{id}`, `/assets/{id}/relationships`, `/assets/{id}/history`, `/maintenance?asset_id={id}` | Asset 360° view |
| `GET /bff/tenants/{id}/usage` | `/tenants/{id}/usage`, `/billing/subscriptions`, `/assets/count` | Tenant admin panel |
| `POST /bff/assets/bulk-action` | Multiple `/assets/{id}/state` | Bulk operations |

### 10.3 BFF Response Shape (Asset Detail)

```json
{
  "asset": {
    "id": "ast_123456",
    "name": "Dell PowerEdge R750",
    "asset_type": "SERVER",
    "state": "ACTIVE",
    "specifications": {
      "cpu": { "model": "Intel Xeon Gold 6348", "cores": 28 },
      "memory": { "total_gb": 256, "type": "DDR5" },
      "storage": [{ "device": "nvme0", "capacity_gb": 1920, "type": "NVMe" }]
    },
    "owner": { "id": "usr_001", "name": "John Doe", "email": "john@company.com" },
    "location": { "datacenter": "DC-EU-01", "rack": "R42", "u_position": 10 },
    "compliance": { "score": 95, "findings_count": 2 },
    "financial": { "purchase_cost": 45000, "current_value": 32000 },
    "relationships": {
      "depends_on": ["ast_001", "ast_002"],
      "hosts": ["ast_100", "ast_101"],
      "connects_to": ["ast_200"]
    }
  },
  "permissions": {
    "can_edit": true,
    "can_delete": false,
    "can_assign": true,
    "can_retire": true
  }
}
```

---

## 11. OpenAPI Specification Structure

### 11.1 File Organization

```
api/contracts/
├── openapi.yaml              # Root spec (includes all)
├── components/
│   ├── schemas.yaml          # Reusable schemas (200+)
│   ├── parameters.yaml       # Common parameters
│   ├── responses.yaml        # Standard responses
│   ├── security.yaml         # Auth schemes
│   └── examples.yaml         # Example values
├── paths/
│   ├── auth.yaml             # /auth/*
│   ├── assets.yaml           # /assets/*
│   ├── tenants.yaml          # /tenants/*
│   ├── billing.yaml          # /billing/*
│   ├── notifications.yaml    # /notifications/*
│   ├── audit.yaml            # /audit/*
│   ├── lansweeper.yaml       # /lansweeper/*
│   └── platform.yaml         # /platform/* (admin)
├── webhooks/
│   ├── webhook-events.yaml   # Event schemas
│   └── webhook-delivery.yaml # Delivery schemas
└── tags.yaml                 # Tag definitions
```

### 11.2 Schema Example (Asset)

```yaml
# api/contracts/components/schemas.yaml
Asset:
  type: object
  required: [id, tenant_id, asset_type, name, state, created_at, updated_at]
  properties:
    id:
      type: string
      format: uuid
      pattern: '^ast_[a-z0-9]{12}$'
      example: "ast_0192f0c123a4"
    tenant_id:
      type: string
      format: uuid
      pattern: '^tn_[a-z0-9]{12}$'
    asset_type:
      type: string
      enum: [SERVER, LAPTOP, NETWORK_DEVICE, VM, CONTAINER, CLOUD_RESOURCE, SOFTWARE, PERIPHERAL, OTHER]
    subtype:
      type: string
      nullable: true
      maxLength: 50
    name:
      type: string
      minLength: 1
      maxLength: 255
    serial_number:
      type: string
      nullable: true
      maxLength: 100
      pattern: '^[A-Za-z0-9\-_]+$'
    asset_tag:
      type: string
      nullable: true
      pattern: '^NXA-[A-Z0-9]{8}$'
    state:
      type: string
      enum: [DISCOVERED, AUTHORIZED, ACTIVE, MAINTENANCE, RETIRED, DISPOSED]
    specifications:
      $ref: '#/components/schemas/AssetSpecifications'
    owner_id:
      type: string
      format: uuid
      pattern: '^usr_[a-z0-9]{12}$'
      nullable: true
    location_id:
      type: string
      format: uuid
      pattern: '^loc_[a-z0-9]{12}$'
      nullable: true
    tags:
      type: array
      items:
        $ref: '#/components/schemas/Tag'
    custom_fields:
      type: object
      additionalProperties: true
    confidence_score:
      type: number
      format: float
      minimum: 0
      maximum: 1
    created_at:
      type: string
      format: date-time
    updated_at:
      type: string
      format: date-time
```

---

## 12. Core API Endpoints

### 12.1 Authentication (`/api/v1/auth`)

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| POST | `/login` | Username/password + MFA | None |
| POST | `/mfa/verify` | Verify TOTP/WebAuthn | Partial (pre-auth) |
| POST | `/refresh` | Refresh access token | Refresh token |
| POST | `/logout` | Revoke refresh token | Access token |
| GET | `/sso/saml` | Initiate SAML login | None |
| POST | `/sso/acs` | SAML Assertion Consumer | None |
| GET | `/sso/oidc` | Initiate OIDC login | None |
| POST | `/sso/callback` | OIDC callback | None |
| POST | `/api-keys` | Create API key | Access token |
| GET | `/api-keys` | List API keys | Access token |
| DELETE | `/api-keys/{id}` | Revoke API key | Access token |

### 12.2 Assets (`/api/v1/assets`)

| Method | Path | Description | Permissions |
|--------|------|-------------|-------------|
| GET | `/` | List assets (paginated, filtered) | `assets:read` |
| POST | `/` | Register asset | `assets:write` |
| GET | `/{id}` | Get asset details | `assets:read` |
| PATCH | `/{id}` | Update asset | `assets:write` |
| DELETE | `/{id}` | Delete asset (soft) | `assets:delete` |
| POST | `/{id}/authorize` | Authorize asset (DISCOVERED→AUTHORIZED) | `assets:authorize` |
| POST | `/{id}/activate` | Activate asset (AUTHORIZED→ACTIVE) | `assets:activate` |
| POST | `/{id}/assign` | Assign asset to user/location | `assets:assign` |
| POST | `/{id}/relocate` | Relocate asset | `assets:relocate` |
| POST | `/{id}/maintenance` | Start maintenance | `assets:maintenance` |
| POST | `/{id}/retire` | Retire asset | `assets:retire` |
| POST | `/{id}/dispose` | Dispose asset | `assets:dispose` |
| POST | `/{id}/restore` | Restore retired asset | `assets:restore` |
| GET | `/{id}/relationships` | Get CI relationships | `assets:read` |
| POST | `/{id}/relationships` | Add CI relationship | `assets:write` |
| DELETE | `/{id}/relationships/{target_id}` | Remove CI relationship | `assets:write` |
| GET | `/{id}/history` | Get asset history/audit trail | `assets:read` |
| GET | `/{id}/compare` | Compare asset versions | `assets:read` |
| POST | `/bulk-import` | Bulk import assets (CSV/Excel) | `assets:write` |
| POST | `/bulk-action` | Bulk state transitions | `assets:write` |

### 12.3 Tenants (`/api/v1/tenants`)

| Method | Path | Description | Permissions |
|--------|------|-------------|-------------|
| POST | `/` | Create tenant (self-service) | None (public) |
| GET | `/{id}` | Get tenant details | `tenants:read` |
| PATCH | `/{id}` | Update tenant settings | `tenants:write` |
| GET | `/{id}/usage` | Get usage/quotas | `tenants:read` |
| POST | `/{id}/suspend` | Suspend tenant | `platform:admin` |
| POST | `/{id}/reactivate` | Reactivate tenant | `platform:admin` |
| POST | `/{id}/offboard` | Initiate offboarding | `platform:admin` |
| POST | `/{id}/domain` | Configure custom domain | `tenants:write` |
| GET | `/{id}/users` | List tenant users | `tenants:read` |
| POST | `/{id}/users` | Invite user | `tenants:write` |
| DELETE | `/{id}/users/{user_id}` | Remove user | `tenants:write` |

### 12.4 Lansweeper (`/api/v1/lansweeper`)

| Method | Path | Description | Permissions |
|--------|------|-------------|-------------|
| POST | `/sites` | Configure Lansweeper site | `lansweeper:write` |
| GET | `/sites` | List sites | `lansweeper:read` |
| GET | `/sites/{id}` | Get site details | `lansweeper:read` |
| PATCH | `/sites/{id}` | Update site config | `lansweeper:write` |
| POST | `/sites/{id}/sync` | Trigger manual sync | `lansweeper:write` |
| GET | `/sites/{id}/sync-history` | Get sync history | `lansweeper:read` |
| GET | `/sites/{id}/field-ownership` | Get field ownership matrix | `lansweeper:read` |
| PATCH | `/sites/{id}/field-ownership` | Update field ownership | `lansweeper:write` |
| GET | `/conflicts` | List reconciliation conflicts | `lansweeper:read` |
| POST | `/conflicts/{id}/resolve` | Resolve conflict | `lansweeper:write` |

### 12.5 Billing (`/api/v1/billing`)

| Method | Path | Description | Permissions |
|--------|------|-------------|-------------|
| GET | `/plans` | List subscription plans | `billing:read` |
| POST | `/subscriptions` | Purchase subscription | `billing:write` |
| GET | `/subscriptions` | List subscriptions | `billing:read` |
| GET | `/subscriptions/{id}` | Get subscription details | `billing:read` |
| PATCH | `/subscriptions/{id}` | Change plan/quantity | `billing:write` |
| POST | `/subscriptions/{id}/cancel` | Cancel subscription | `billing:write` |
| GET | `/invoices` | List invoices | `billing:read` |
| GET | `/invoices/{id}` | Get invoice (with PDF link) | `billing:read` |
| GET | `/allocation-rules` | List allocation rules | `billing:read` |
| POST | `/allocation-rules` | Create allocation rule | `billing:write` |
| GET | `/contracts` | List contracts | `billing:read` |
| POST | `/contracts` | Create contract | `billing:write` |

### 12.6 Audit & Compliance (`/api/v1/audit`)

| Method | Path | Description | Permissions |
|--------|------|-------------|-------------|
| GET | `/frameworks` | List compliance frameworks | `audit:read` |
| POST | `/frameworks` | Create custom framework | `audit:write` |
| GET | `/assessments` | List assessments | `audit:read` |
| POST | `/assessments` | Run assessment | `audit:write` |
| GET | `/assessments/{id}` | Get assessment results | `audit:read` |
| GET | `/assessments/{id}/evidence` | Download evidence package | `audit:read` |
| GET | `/findings` | List findings (filterable) | `audit:read` |
| GET | `/risks` | List risks | `audit:read` |
| POST | `/risks` | Create risk | `audit:write` |
| GET | `/dashboard` | Compliance dashboard | `audit:read` |

---

## 13. Developer Portal & SDKs

### 13.1 Developer Portal Features

| Feature | Description |
|---------|-------------|
| **OpenAPI Explorer** | Interactive API docs with try-it-out |
| **Authentication Playground** | Test JWT, API keys, SSO flows |
| **SDK Generation** | TypeScript, Python, Go, C# |
| **API Key Management** | Create, rotate, revoke keys |
| **Usage Analytics** | Request volume, latency, errors per key |
| **Webhook Testing** | ngrok-style tunnel for local development |
| **Changelog** | Version history, deprecation notices |

### 13.2 SDK Installation

```bash
# TypeScript/JavaScript
npm install @nexusasset/sdk

# Python
pip install nexusasset-sdk

# Go
go get github.com/nexusasset/sdk-go

# C#
dotnet add package NexusAsset.Sdk
```

### 13.3 SDK Usage Example (TypeScript)

```typescript
import { NexusAssetClient } from '@nexusasset/sdk';

const client = new NexusAssetClient({
  baseUrl: 'https://api.nexusasset.cloud/api/v1',
  apiKey: 'nxa_ak_...',  // or use OAuth
  tenantId: 'tn_abc123'  // optional if in JWT
});

// List assets with filtering
const assets = await client.assets.list({
  filter: { asset_type: ['SERVER', 'LAPTOP'] },
  sort: '-created_at',
  limit: 50
});

// Get asset detail with relationships
const asset = await client.assets.get('ast_123456', {
  include: ['relationships', 'history', 'compliance']
});

// Create asset with idempotency
const newAsset = await client.assets.create({
  name: 'New Server',
  asset_type: 'SERVER',
  serial_number: 'SN123456',
}, {
  idempotencyKey: 'idem_0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6'
});

// Subscribe to webhooks
await client.webhooks.create({
  url: 'https://myapp.com/webhooks/nexusasset',
  events: ['asset.created', 'asset.state_changed'],
  secret: 'whsec_...'
});
```

---

## 14. API Governance & Compliance

### 14.1 API Review Process

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              API GOVERNANCE WORKFLOW                                                │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  1. PROPOSAL                                                                                        │
│     Developer creates API change PR → Updates openapi.yaml                                          │
│                                                                                                     │
│  2. AUTOMATED CHECKS (CI)                                                                           │
│     □ openapi-spec-validator (OpenAPI 3.1 compliance)                                              │
│     □ spectral (style guide: naming, descriptions, examples)                                       │
│     □ breaking-change-detector (vs. current production spec)                                       │
│     □ schema-compatibility (request/response backward compatible)                                  │
│     □ security-scan (no PII in paths, proper auth on all endpoints)                               │
│                                                                                                     │
│  3. MANUAL REVIEW                                                                                    │
│     □ API Governance Board (weekly)                                                                 │
│     □ Security review (for new auth scopes, PII exposure)                                          │
│     □ Performance review (new endpoints: load test plan)                                           │
│     □ Documentation review (descriptions, examples, error codes)                                   │
│                                                                                                     │
│  4. APPROVAL & MERGE                                                                                 │
│     □ 2 approvals from Board members                                                                │
│     □ Merged to `main` → triggers spec publish to portal                                           │
│                                                                                                     │
│  5. DEPLOYMENT                                                                                        │
│     □ Staging deployment (auto) → Contract tests run                                               │
│     □ Production deployment (ArgoCD promotion)                                                     │
│     □ Post-deploy validation (smoke tests, synthetic monitoring)                                   │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 14.2 Contract Testing

```yaml
# Provider contract test (backend)
# tests/contracts/provider/asset_api_test.py
def test_asset_creation_contract():
    """Provider honors contract for POST /assets"""
    response = client.post("/api/v1/assets", json=valid_asset_payload)
    assert response.status_code == 201
    assert_response_matches_schema(response, "Asset")

# Consumer contract test (frontend/BFF)
# tests/contracts/consumer/bff_asset_test.py
def test_bff_consumes_asset_contract():
    """BFF can consume asset response"""
    mock_response = load_fixture("asset_response.json")
    result = bff_client.transform_asset(mock_response)
    assert result.ui_fields == expected_ui_fields
```

### 14.3 API Metrics & SLIs

| SLI | Target | Measurement |
|-----|--------|-------------|
| **Availability** | 99.99% | Successful requests / Total requests (5xx excluded) |
| **Latency (p99)** | < 500ms | `histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))` |
| **Error Rate** | < 0.1% | `rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])` |
| **Throughput** | > 10k RPS | `rate(http_requests_total[5m])` |
| **Contract Compliance** | 100% | Spectral validation pass rate in CI |

---

## 15. Diagrams

| Diagram ID | Title | Type | Location | Description |
|------------|-------|------|----------|-------------|
| **DIA-069** | Authentication Flows | Sequence | Section 2.2 | Login, SSO, API Key, Refresh |
| **DIA-070** | Tenant Context Resolution | Flowchart | Section 3.1 | Subdomain → Header → JWT → Default |
| **DIA-071** | Authorization Layers | Layered | Section 2.3 | Transport → Auth → Tenant → RBAC → ABAC |
| **DIA-072** | Versioning Strategy | Component | Section 4.1 | URL path, headers, lifecycle |
| **DIA-073** | Request/Response Envelope | Component | Section 5.3 | Data, meta, links structure |
| **DIA-074** | Error Format (RFC 9457) | Component | Section 6.1 | Problem Details structure |
| **DIA-075** | Cursor Pagination | Sequence | Section 7.1 | First → Cursor → Next requests |
| **DIA-076** | Webhook Delivery | Sequence | Section 9 | Register → Event → Deliver → Retry → DLQ |
| **DIA-077** | BFF Aggregation | Sequence | Section 10.2 | Frontend → BFF → Multiple Backend → Aggregate |
| **DIA-078** | OpenAPI Spec Structure | Package | Section 11.1 | File organization |
| **DIA-079** | API Governance Workflow | Flowchart | Section 14.1 | Proposal → CI → Review → Approve → Deploy |
| **DIA-080** | SDK Architecture | Component | Section 13 | Client → HTTP → Retry → Auth → Types |

---

## 16. Tables

| Table ID | Title | Location | Row Count |
|----------|-------|----------|-----------|
| **TBL-154** | API Design Principles | Section 1.1 | 10 |
| **TBL-155** | API Style Guide | Section 1.2 | 12 |
| **TBL-156** | JWT Claims | Section 2.1 | 12 |
| **TBL-157** | Authorization Levels | Section 2.3 | 6 |
| **TBL-158** | Tenant Context Priority | Section 3.1 | 4 |
| **TBL-159** | Tenant Context Headers | Section 3.2 | 3 |
| **TBL-160** | Version Lifecycle | Section 4.2 | 5 |
| **TBL-161** | Standard Request Headers | Section 5.1 | 7 |
| **TBL-162** | Standard Response Headers | Section 5.2 | 6 |
| **TBL-163** | Error Types (RFC 9457) | Section 6.2 | 9 |
| **TBL-164** | Error Codes | Section 6.3 | 14 |
| **TBL-165** | Rate Limit Tiers | Section 8.1 | 4 |
| **TBL-166** | BFF Endpoints | Section 10.2 | 4 |
| **TBL-167** | Auth Endpoints | Section 12.1 | 10 |
| **TBL-168** | Asset Endpoints | Section 12.2 | 20 |
| **TBL-169** | Tenant Endpoints | Section 12.3 | 10 |
| **TBL-170** | Lansweeper Endpoints | Section 12.4 | 9 |
| **TBL-171** | Billing Endpoints | Section 12.5 | 10 |
| **TBL-172** | Audit Endpoints | Section 12.6 | 10 |
| **TBL-173** | Developer Portal Features | Section 13.1 | 7 |
| **TBL-174** | SDK Languages | Section 13.2 | 4 |
| **TBL-175** | API Governance Checks | Section 14.1 | 15 |
| **TBL-176** | API SLIs | Section 14.3 | 5 |
| **TBL-177** | Diagram Index | Section 15 | 12 |
| **TBL-178** | Table Index | Section 16 | 25 |

---

---

**End of DOC-07 — NexusAsset Cloud API Design**

*Document Control: Version 1.0.0 | Classification: Internal — Confidential | Next Review: 2026-12-09*

*For questions or updates: docs@nexusasset.cloud | API Governance Board: api-governance@nexusasset.cloud*