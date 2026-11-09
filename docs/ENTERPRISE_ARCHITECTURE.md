# ═══════════════════════════════════════════════════════════════
#  NEXUSASSET CLOUD — ENTERPRISE ARCHITECTURE
# ═══════════════════════════════════════════════════════════════
#  Document ID        : EA-001
#  Version            : 1.0.0
#  Status             : Baseline (Target/Design — Unverified Implementation)
#  Classification     : Internal — Architecture & Engineering
#  Owner              : Architecture Guild / Platform Engineering
#  Last Reviewed      : 2026-11-09
#  Next Review        : 2026-12-09
#  Source Evidence    : docs/02-design/*, docs/12-adr/*, prompts/*, platform/**/*.description.md
#  Canonical Repo     : nexusasset-cloud/
#  Domains            : app.nexusasset.cloud  |  api.nexusasset.cloud
#  Package Scope      : @nexusasset/*
#  Secret Prefix      : NXA_*
# ═══════════════════════════════════════════════════════════════

# NexusAsset Cloud — Enterprise Architecture
**Where Discovery Meets Enterprise**

> **Purpose** — Define the target architecture for NexusAsset Cloud: system boundaries, containers, components, data, contracts, integration patterns, tenancy, events, and key flows. This document consolidates all design artifacts into a single authoritative reference. All sections are **Target/Design** unless explicitly marked **Verified**.

---

## Table of Contents
1. [Architectural Principles](#1-architectural-principles)
2. [System Context (C4 Level 1)](#2-system-context-c4-level-1)
3. [Container Topology (C4 Level 2)](#3-container-topology-c4-level-2)
4. [Component Architecture (C4 Level 3)](#4-component-architecture-c4-level-3)
5. [Clean Architecture & Dependency Rules](#5-clean-architecture--dependency-rules)
6. [Data Model, ERD & RLS](#6-data-model-erd--rls)
7. [API Contract & Standards](#7-api-contract--standards)
8. [Backend-for-Frontend (BFF) Pattern](#8-backend-for-frontend-bff-pattern)
9. [Lansweeper Integration Architecture](#9-lansweeper-integration-architecture)
10. [Multi-Tenancy Strategy](#10-multi-tenancy-strategy)
11. [Event-Driven & Asynchronous Processing](#11-event-driven--asynchronous-processing)
12. [Key Sequence Flows](#12-key-sequence-flows)
13. [Current vs Target State Matrix](#13-current-vs-target-state-matrix)
14. [Governance, ADRs & Compliance](#14-governance-adrs--compliance)
15. [Evidence & Verification Requirements](#15-evidence--verification-requirements)

---

## 1. Architectural Principles

┌─────────────────────────────────────────────────────────────────────┐
│ ▸ PRINCIPLE 1 — Clean Architecture                                   │
│   Dependency direction:  Interfaces → Application → Domain           │
│   Infrastructure implements ports/adapters; Domain has ZERO         │
│   framework imports (no Django, ORM, HTTP, Celery, UI, SDKs).       │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ PRINCIPLE 2 — Domain-Driven Design (DDD)                          │
│   Bounded contexts (`nxa_accounts`, `nxa_tenants`, `nxa_assets`,     │
│   `nxa_lansweeper`, `nxa_billing`, `nxa_notifications`, `nxa_audit`)│
│   own ubiquitous language, invariants, and business rules.          │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ PRINCIPLE 3 — OpenAPI-First                                       │
│   `api/contracts/openapi.yaml` is the **single source of truth**.   │
│   Code generation, contract tests, and BFF alignment mandatory.     │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ PRINCIPLE 4 — BFF for Web                                          │
│   `nxa-frontend`/BFF aggregates backend calls, manages browser-safe │
│   auth/session, shapes UI responses. MUST NOT own business logic.   │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ PRINCIPLE 5 — Multi-Tenancy by Design                             │
│   Tenant context resolution + application authorization +           │
│   PostgreSQL 16 Row-Level Security (RLS) as defense-in-depth.       │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ PRINCIPLE 6 — Idempotent Integrations                             │
│   Lansweeper sync (GraphQL/webhooks/delta/bulk) and all async       │
│   operations tolerate retries/replay without corruption.            │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ PRINCIPLE 7 — Observability & Auditability                        │
│   Significant state changes emit structured telemetry (metrics,     │
│   traces, logs) and immutable audit evidence (`nxa_audit`).         │
└─────────────────────────────────────────────────────────────────────┘

---

## 2. System Context (C4 Level 1)

### 2.1 Actors & External Systems

| # | Actor / System | Type | Interaction | Trust Boundary | Auth/Validation |
|---|---|---|---|---|---|
| 1 | **Web Users** (Admin, Manager, Auditor, Operator) | Human | `app.nexusasset.cloud` → `nxa-frontend`/BFF | Public Internet → Edge | JWT/OIDC, tenant-scoped RBAC |
| 2 | **Mobile Users** (Approved flows) | Human | Mobile app → API Gateway | Public Internet → Edge | Secure token handling, mTLS where required |
| 3 | **Lansweeper Platform** | External System | GraphQL / Webhooks / Bulk / Delta sync | **External Trust Boundary** | API key / OAuth, idempotent verification, Field Ownership Matrix |
| 4 | **Email / SMS / Push Providers** | External System | Outbound notifications | Outbound Integration | `NXA_*` secrets, provider APIs |
| 5 | **CI/CD / Deployment Systems** | Automation | Build / Release / Deploy | **Privileged Boundary** | Approval gates, audit trail, least-privilege runners |

### 2.2 Trust Boundaries (ASCII)

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                              PUBLIC INTERNET                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐   │
│  │ Web Users   │  │Mobile Users │  │ Lansweeper  │  │  CI/CD Systems  │   │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └────────┬────────┘   │
└─────────┼────────────────┼────────────────┼───────────────────┼────────────┘
          │                │                │                   │
          ▼                ▼                ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          EDGE: nxa-traefik-public / nxa-le                  │
│         TLS Termination │ Rate Limiting │ WAF │ Cert Management             │
└────────────────────────────┬────────────────────────────────────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         ▼                   ▼                   ▼
┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│ nxa-frontend   │  │ nxa-backend    │  │  nxa-worker    │
│ (Next.js +     │  │ (Django/DRF    │  │  (Celery       │
│  BFF)          │  │  API)          │  │   Workers)     │
└───────┬────────┘  └───────┬────────┘  └───────┬────────┘
        │                   │                   │
        │    ┌──────────────┼──────────────┐    │
        │    ▼              ▼              ▼    │
        │ ┌──────┐    ┌──────────┐    ┌──────┐  │
        │ │nxa-db│    │ nxa-redis │    │ ...  │  │
        │ │(Postgres)  │ (Cache/   │    │      │  │
        │ │ + RLS)     │  Broker)  │    │      │  │
        │ └──────┘    └──────────┘    └──────┘  │
        └────────────────────────────────────────┘
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
           ┌──────────────┐    ┌──────────────┐
           │  External    │    │ Notification │
           │  Integrations│    │  Providers   │
           │ (Lansweeper) │    │ (Email/SMS)  │
           └──────────────┘    └──────────────┘
```

### 2.3 Boundary Rules

> **Note** — Every boundary crossing requires:
> - ✅ Authentication & Authorization
> - ✅ Input validation & schema enforcement
> - ✅ Secret protection (`NXA_*` via Vault/secrets manager)
> - ✅ Observability (correlation IDs, structured logs, metrics)
> - ✅ Audit trail for privileged operations

---

## 3. Container Topology (C4 Level 2)

### 3.1 Runtime Containers (Target)

| Container | Technology | Responsibility | Key Dependencies | Status |
|---|---|---|---|---|
| **nxa-frontend** | Next.js 15 + React 19 | UI, BFF boundary, auth/session config | `nxa-backend` (API), Auth provider | 🟡 Descriptor/Target |
| **nxa-backend** | Django 5 + DRF | REST API, use cases, domain orchestration | PostgreSQL, Redis, Integrations | 🟡 Descriptor/Target |
| **nxa-worker** | Celery (Redis broker) | Async jobs: sync, notifications, scheduled | Redis, DB, External APIs | 🟡 Descriptor/Target |
| **nxa-db** | PostgreSQL 16 | Primary data store, RLS enforcement | Storage, Backup, Replication | 🟡 Descriptor/Target |
| **nxa-redis** | Redis 7 | Cache, Celery broker, session store | Memory, Persistence config | 🟡 Descriptor/Target |
| **nxa-traefik-public** | Traefik v3 | Edge routing, TLS termination, rate limiting | Certs (Let's Encrypt), DNS, Service mesh | 🟡 Descriptor/Target |
| **nxa-le** | Cert-manager/LE | Automated certificate provisioning | DNS challenge, ACME | 🟡 Descriptor/Target |

### 3.2 Runtime Concerns (Target)

| Concern | Requirement | Evidence Needed |
|---|---|---|
| Health/Readiness | `/healthz`, `/readyz` per container | Implemented endpoints + probes |
| Migrations | Zero-downtime, backward-compatible | Migration scripts + CI gate |
| Worker Scaling | Horizontal, queue-depth aware | KEDA/HPA config + load tests |
| Secret Injection | Vault/SealedSecrets, no plaintext in images | Secret scan + runtime audit |
| Tenant-Safe Config | No cross-tenant leakage in shared config | Config review + negative tests |
| Logging/Metrics/Traces | OpenTelemetry → Prometheus/Grafana/Loki | OTel SDK + dashboards |
| Backup/Restore | RPO/RTO defined, tested quarterly | Runbook + drill evidence |

---

## 4. Component Architecture (C4 Level 3)

### 4.1 Backend Bounded Contexts — Layered Structure

Each bounded context follows **Clean Architecture** layering:

```
backend/nexusasset/
├── nxa_accounts/        # Identity, authentication, authorization
├── nxa_tenants/         # Tenancy, membership, hierarchy
├── nxa_assets/          # CMDB, asset lifecycle, relationships
├── nxa_lansweeper/      # Discovery sync, conflict resolution
├── nxa_billing/         # Commercial, subscriptions, entitlements
├── nxa_notifications/   # Multi-channel delivery, templates
├── nxa_audit/           # Immutable audit trail, query
└── nxa_core/            # Shared kernel, cross-cutting concerns
```

### 4.2 Per-Context Layer Map (Target)

| Bounded Context | Domain | Application | Infrastructure | Interfaces |
|---|---|---|---|---|
| **nxa_accounts** | Identity, User, Role, Permission concepts | Auth use cases, User management, Ports | Django ORM adapters, External IdP adapters | REST Auth APIs, Token introspection |
| **nxa_tenants** | Tenant, Membership, Hierarchy concepts | Tenant context resolution, Admin use cases | RLS policy repos, Tenant cache | Tenant CRUD, Context middleware |
| **nxa_assets** | Asset, Relationship, Lifecycle, Category | CMDB use cases, Search, Bulk ops | Search index (Postgres/ES), Repos | Asset REST, GraphQL (optional) |
| **nxa_lansweeper** | Discovery, Sync, Conflict, Field Ownership | Sync orchestration, Webhook handling, Mapping policy | Lansweeper GraphQL client, Job store, Mappers | Sync status APIs, Webhook endpoints |
| **nxa_billing** | Subscription, Plan, Entitlement, Invoice | Billing cycles, Provisioning, Proration | Payment gateway adapters, Ledger | Billing APIs, Webhooks |
| **nxa_notifications** | Notification, Template, Channel, Preference | Dispatch, Rendering, Retry policy | Email/SMS/Push providers, Template engine | Notification APIs, Preferences |
| **nxa_audit** | AuditEvent, Actor, Resource, Outcome | Record, Query, Retention, Export | Append-only store, Search index | Audit query APIs, Export jobs |
| **nxa_core** | Shared value objects, Exceptions, Events | Cross-cutting use cases (health, config) | Logging, Metrics, Tracing, Config | Internal only |

### 4.3 Frontend Components (Target)

| Area | Components |
|---|---|
| **App Shell** | Routes, Layout, Navigation, Theme provider |
| **Auth/Session** | Login/SSO flow, Token storage, Refresh, CSRF |
| **BFF Handlers** | Aggregation endpoints, Response shaping, Error normalization |
| **API Client** | Typed client (OpenAPI codegen), Interceptors, Retry |
| **Feature Modules** | Assets (list/detail/bulk), Integrations (Lansweeper), Audit, Admin |
| **Shared UI** | Design system tokens, Components, Forms, Tables, Charts |

---

## 5. Clean Architecture & Dependency Rules

### 5.1 Layer Dependency Matrix

| Layer | MAY Depend On | MUST NOT Depend On |
|---|---|---|
| **Domain** | Std lib primitives, Domain-only abstractions | Django, DRF, ORM, HTTP, Celery, UI, External SDKs |
| **Application** | Domain, Ports/Interfaces, DTOs | Concrete ORM, Web framework, DB drivers |
| **Infrastructure** | Application ports, Domain (for mapping), Frameworks/SDKs | Owning business rules |
| **Interfaces** | Application/Use cases, DTOs/Schemas | Direct domain persistence shortcuts bypassing application |

### 5.2 Enforcement Mechanisms (Target)

| Mechanism | Tool/Method | CI Gate |
|---|---|---|
| **Import Linter** | `import-linter` / `python-architecture-linter` | Fail on violation |
| **Architecture Tests** | Custom pytest suites per bounded context | Required in pipeline |
| **Code Review Checklist** | PR template: "Dependency rule verified" | Required approval |
| **ADR for Exceptions** | `docs/12-adr/NNNN-exception-description.md` | Architecture Guild sign-off |

### 5.3 NexusAsset-Specific Applications

- **Lansweeper Client** → Infrastructure (GraphQL HTTP client)
- **Sync Decision Logic** (mapping precedence, field ownership, conflict policy) → **Domain/Application** (NOT infrastructure)
- **Tenant/AuthZ Checks** → Enforced in Application + Interface + Data layers (defense in depth)

> **Warning** — Any domain model importing `django.db`, `rest_framework`, `celery`, or external SDKs is a **Clean Architecture violation** and must be refactored or documented via ADR.

---

## 6. Data Model, ERD & RLS

### 6.1 Data Modeling Requirements (Target)

| Requirement | Detail | Verification |
|---|---|---|
| **Tenant Scope** | All tenant-owned tables include `tenant_id` (UUID) + RLS policy | Migration review + RLS test |
| **Identity/Integrity** | PKs, FKs, unique constraints, nullability reflect domain invariants | Schema diff + constraint tests |
| **Provenance** | Discovery-sourced records retain `source_id`, `source_system`, `last_synced_at` | Column audit + sync tests |
| **Auditability** | Sensitive changes → `nxa_audit` events (actor, resource, before/after) | Audit log review |
| **Performance** | Indexes on: `tenant_id`, common lookups, FKs, sync cursors, external IDs | `EXPLAIN ANALYZE` + index advisor |
| **Retention** | Per-data-class policy (legal, operational, analytics) | Retention job + compliance sign-off |

### 6.2 Conceptual Entities (Target)

```
Tenant ◄──┐
          ├── User ◄── Membership ──► Role/Permission
          │
          ├── Asset ◄── AssetRelationship
          │         │
          │         ├── DiscoveryRecord (Lansweeper provenance)
          │         └── SyncJob / SyncCursor / SyncConflict
          │
          ├── Notification ◄── Template ◄── Channel
          │
          ├── BillingRecord (Subscription, Invoice, Entitlement)
          │
          └── AuditEvent (append-only, indexed by tenant+timestamp)
```

### 6.3 RLS Implementation Notes (Target)

| Aspect | Requirement |
|---|---|
| **Policy Scope** | All tenant tables: `CREATE POLICY ... USING (tenant_id = current_setting('app.current_tenant')::uuid)` |
| **Session Context** | Middleware sets `SET LOCAL app.current_tenant = '<uuid>'` per request |
| **Admin Bypass** | Superuser role or `app.bypass_rls = 'on'` (audited, time-limited) |
| **Testing** | Negative tests: cross-tenant query returns 0 rows; positive tests: correct isolation |
| **Defense-in-Depth** | RLS **supplements** (not replaces) application-level authorization |

> **Note** — No verified schema dump or migrations exist in current repository state. This section defines **target requirements**.

---

## 7. API Contract & Standards

### 7.1 OpenAPI 3.1 — Source of Truth

| Artifact | Path | Status |
|---|---|---|
| **OpenAPI Spec** | `api/contracts/openapi.yaml` | 🟡 Descriptor/Skeleton (target) |
| **Descriptor Evidence** | `platform/api/contracts/openapi.yaml.description.md` | ✅ Exists |
| **Codegen Output** | `packages/@nexusasset/api-client/`, `frontend/src/api/` | 🟡 Pending |

### 7.2 API Standards (Target)

| Standard | Requirement |
|---|---|
| **Versioning** | URL path versioning (`/api/v1/...`); breaking changes = new version |
| **Naming** | kebab-case resources, plural nouns (`/assets`, `/sync-jobs`) |
| **Operation IDs** | `{context}.{action}` (e.g., `assets.list`, `lansweeper.sync.trigger`) |
| **Tags** | Bounded context alignment (`nxa-assets`, `nxa-lansweeper`, ...) |
| **Auth** | Explicit `security:` per operation; tenant scope in `x-tenant-scope` extension |
| **Pagination** | Cursor-based (`cursor`, `limit`); `Link` header for next/prev |
| **Filtering/Sorting** | Standard query params (`filter[field]=value`, `sort=-created_at`) |
| **Error Envelope** | RFC 9457 `ProblemDetails` + `x-request-id` correlation |
| **Idempotency** | `Idempotency-Key` header for `POST`/`PATCH` on sync/trigger ops |
| **Deprecation** | `deprecated: true` + `Sunset` header + 6-month notice |

### 7.3 BFF vs Public/Internal API

| Characteristic | BFF (Web) | Public/Mobile API |
|---|---|---|
| **Aggregation** | Allowed (multi-resource responses) | Single-resource, contract-stable |
| **Auth** | Browser session + CSRF | Bearer token (JWT/Opaque) |
| **Contract** | Internal, may evolve with UI | Versioned, governed, documented |
| **Authorization** | Propagates tenant context | Enforces tenant scope independently |

---

## 8. Backend-for-Frontend (BFF) Pattern

### 8.1 Responsibility Matrix (Target)

| Responsibility | BFF (`nxa-frontend`) | Backend (`nxa-backend`) |
|---|---|---|
| **UI Aggregation** | ✅ Yes | Optional (internal services) |
| **Browser Auth/Session** | ✅ Yes (cookies, refresh, CSRF) | Token validation, User info, **Final AuthZ authority** |
| **Business Rules** | ❌ No | ✅ Yes (Domain/Application) |
| **Tenant Authorization** | Must enforce/propagate correctly | ✅ Final authority |
| **Contract Alignment** | Must align with OpenAPI/backend | ✅ Source of domain/API behavior |

### 8.2 Risks & Mitigations (Target)

| Risk | Mitigation |
|---|---|
| Business logic duplication | Architecture tests; PR checklist; code review |
| Token/sensitive data leakage | No tokens in browser storage; HttpOnly cookies; CSP |
| Bypassing backend AuthZ | Negative tests: BFF call without tenant → 403 |
| Contract divergence | Contract tests in CI; OpenAPI codegen for BFF client |

### 8.3 Verification Checklist (Target)

- [ ] BFF auth/CSRF/CORS tests pass
- [ ] Contract tests (BFF ↔ Backend) pass
- [ ] Tenant-negative tests pass (cross-tenant → 403/404)
- [ ] Security review completed

---

## 9. Lansweeper Integration Architecture

### 9.1 Integration Overview (Target)

```text
┌─────────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│  Lansweeper Cloud/  │────▶│  nxa-lansweeper      │────▶│  nxa-assets /       │
│  On-Prem (GraphQL)  │     │  (Bounded Context)   │     │  nxa-audit          │
└─────────────────────┘     └──────────────────────┘     └─────────────────────┘
         │                            │
         │ Webhooks                   │ Delta / Bulk / Full Sync
         ▼                            ▼
┌─────────────────────┐     ┌──────────────────────┐
│  nxa-lansweeper     │     │  nxa-worker          │
│  Webhook Endpoint   │────▶│  (Celery Jobs)       │
└─────────────────────┘     └──────────────────────┘
```

### 9.2 Sync Modes (Target)

| Mode | Trigger | Scope | Idempotency Key |
|---|---|---|---|
| **Full Sync** | Scheduled (daily/weekly) | All assets | `full-sync:{tenant}:{timestamp}` |
| **Delta Sync** | Scheduled (hourly) | Changed since last cursor | `delta-sync:{tenant}:{cursor}` |
| **Bulk Import** | Operator/API | CSV/JSON payload | `bulk-import:{tenant}:{batchId}` |
| **Webhook** | Lansweeper event | Single asset change | `webhook:{tenant}:{eventId}` (dedupe) |

### 9.3 Field Ownership Matrix (Target)

| Field Category | Owner | Write Authority | Conflict Policy |
|---|---|---|---|
| **Core Identity** (MAC, Serial, BIOS UUID) | Lansweeper | Lansweeper only | Reject manual override |
| **Discovery Data** (HW specs, SW list, Network) | Lansweeper | Lansweeper only | Last-write-wins (timestamp) |
| **CMDB Enrichment** (Owner, Location, Cost Center, Tags) | **NexusAsset** | NexusAsset users | **Preserve manual edits** |
| **Relationships** (Parent/Child, Dependencies) | **NexusAsset** | NexusAsset users | Preserve manual |
| **Custom Fields** | Configurable | Configurable | Configurable per field |

> **Critical** — Field ownership is **domain logic** (Application layer), NOT infrastructure. The Lansweeper client only fetches/normalizes; mapping policy decides.

### 9.4 Idempotency & Conflict Resolution (Target)

| Mechanism | Implementation |
|---|---|
| **Deduplication** | `source_id` + `source_system` unique constraint per tenant |
| **Upsert Logic** | `ON CONFLICT (tenant_id, source_id) DO UPDATE SET ... WHERE updated_at < EXCLUDED.updated_at` |
| **Conflict Detection** | Compare incoming vs stored; flag if NexusAsset-owned field changed externally |
| **Conflict Queue** | `SyncConflict` entity → Operator review UI → Resolve (accept/discard/merge) |
| **Audit Trail** | Every sync attempt → `nxa_audit` (actor=system, resource=asset, outcome=success/conflict) |

### 9.5 Failure Handling (Target)

| Failure Type | Strategy |
|---|---|
| **Transient (network, 5xx)** | Exponential backoff (max 5 retries), jitter, dead-letter after exhaustion |
| **Auth Failure (401/403)** | Alert ops, pause tenant sync, manual intervention |
| **Schema/Mapping Error** | Quarantine batch, alert, no partial commits |
| **Rate Limit (429)** | Respect `Retry-After`, backoff, resume |
| **Data Corruption Risk** | Transactional boundary per asset; compensating transactions for partial failure |

---

## 10. Multi-Tenancy Strategy

### 10.1 Tenant Context Resolution (Target)

```text
Request → Traefik → BFF/Backend
                │
                ▼
        ┌───────────────────┐
        │ Auth Middleware   │ → Validate JWT / Session
        └─────────┬─────────┘
                  │
                  ▼
        ┌───────────────────┐
        │ Tenant Resolver   │ → Extract tenant_id from:
        │                   │   • JWT claim (preferred)
        │                   │   • Subdomain (app.tenant.nexusasset.cloud)
        │                   │   • Header (X-Tenant-ID, internal only)
        └─────────┬─────────┘
                  │
                  ▼
        ┌───────────────────┐
        │ Authorization     │ → Check membership + role + permission
        │ (Application)     │
        └─────────┬─────────┘
                  │
                  ▼
        ┌───────────────────┐
        │ RLS Context Set   │ → SET LOCAL app.current_tenant = '<uuid>'
        │ (Data Layer)      │
        └───────────────────┘
```

### 10.2 Isolation Layers (Defense in Depth)

| Layer | Mechanism | Verification |
|---|---|---|
| **Application** | Tenant-aware use cases; repository methods auto-filter by `tenant_id` | Unit tests: cross-tenant repo call → empty |
| **Data (RLS)** | PostgreSQL policies on all tenant tables | Integration tests: `SET ROLE tenant_x; SELECT * FROM assets` → only tenant_x |
| **Infrastructure** | Separate connection pools per tenant (optional, high isolation) | Config review |
| **Observability** | `tenant_id` in all structured logs, metrics, traces | Log query: `tenant_id:xxx` shows only that tenant |

### 10.3 Admin / Cross-Tenant Scenarios (Target)

| Scenario | Handling |
|---|---|
| **Platform Admin** | `super_admin` role bypasses tenant filter (audited, time-boxed) |
| **Tenant Admin** | Full access within own tenant only |
| **Support Impersonation** | Explicit consent, audit log, limited duration, read-only by default |
| **Reporting/Analytics** | Separate read-replica with aggregated, anonymized views |

---

## 11. Event-Driven & Asynchronous Processing

### 11.1 Event/Job Requirements (Target)

| Requirement | Specification |
|---|---|
| **Tenant Context** | Every event/job carries validated `tenant_id`; no implicit cross-tenant |
| **Idempotency** | Handlers tolerate at-least-once delivery; deduplication via `event_id` |
| **Schema Versioning** | Event payloads include `event_version`; backward-compatible evolution |
| **Observability** | Correlation ID propagation; structured logs; Prometheus metrics per handler |
| **Failure Handling** | Retry: exponential backoff (1m, 5m, 15m, 1h, 6h); Dead-letter queue after max retries |
| **Audit** | State-changing events → `nxa_audit` record |

### 11.2 Candidate Event Catalog (Target)

| Event | Publisher | Consumers | Payload Essentials |
|---|---|---|---|
| `asset.discovered` | `nxa-lansweeper` | `nxa-assets`, `nxa-audit` | `tenant_id`, `asset_id`, `source_id`, `discovered_at` |
| `asset.synced` | `nxa-lansweeper` | `nxa-assets`, `nxa-notifications` | `tenant_id`, `asset_id`, `changes`, `conflict_flag` |
| `sync.conflict.detected` | `nxa-lansweeper` | `nxa-notifications`, `nxa-audit` | `tenant_id`, `asset_id`, `field`, `local_value`, `remote_value` |
| `audit.recorded` | `nxa-audit` | `nxa-notifications` (alerting) | `tenant_id`, `event_type`, `actor`, `resource`, `outcome` |
| `notification.requested` | Any | `nxa-notifications` | `tenant_id`, `template`, `recipients`, `payload` |
| `billing.entitlement.changed` | `nxa-billing` | `nxa-assets` (feature flags) | `tenant_id`, `entitlement`, `status` |

### 11.3 Broker & Infrastructure (Target)

| Component | Technology | Config Notes |
|---|---|---|
| **Message Broker** | Redis 7 (Streams) / RabbitMQ (if needed) | Persistence, consumer groups, TTL |
| **Task Queue** | Celery (Redis broker) | Priority queues, rate limiting, result backend |
| **Scheduled Jobs** | Celery Beat / Cron | Idempotent triggers, distributed lock |

---

## 12. Key Sequence Flows

> **Standard** — All diagrams use ASCII/text (Mermaid deferred per repository policy). Each flow states: Actors/Components, Preconditions, Main Path, Failure Paths, Status (Target/Verified).

### 12.1 User Login & Authenticated Request

```
User → Traefik: HTTPS GET /app (or /api)
Traefik → BFF: Forward (TLS terminated)
BFF → Auth Provider: OIDC / SSO redirect
Auth Provider → User: Login form
User → Auth Provider: Credentials
Auth Provider → BFF: Authorization code
BFF → Auth Provider: Token exchange
BFF → User: Set HttpOnly cookie (session)
User → BFF: Subsequent request (cookie)
BFF → Backend: API call + Bearer token (server-side)
Backend → AuthZ: Validate token, extract tenant_id, roles
Backend → Application: Use case execution (tenant-scoped)
Application → Repository: Tenant-filtered query
Repository → PostgreSQL: SELECT ... WHERE tenant_id = $1 (RLS)
PostgreSQL → Repository → Application → Backend → BFF → User: Response
```

**Failure Paths**: Invalid token → 401; Expired session → Refresh/re-login; Cross-tenant access → 403; RLS bypass attempt → Audit alert.

---

### 12.2 Asset List/Detail Query

```
User → BFF: GET /api/v1/assets?cursor=...&limit=50
BFF → Backend: GET /api/v1/assets (server-token + tenant)
Backend → AuthZ: Validate principal, tenant membership
Backend → Application: ListAssetsUseCase(tenant, filters, pagination)
Application → Repository: FindByTenant(tenant, spec, cursor, limit)
Repository → PostgreSQL: Index scan on (tenant_id, created_at) + RLS
PostgreSQL → Repository → Application → Backend → BFF → User: Paginated assets
```

**Failure Paths**: Invalid filters → 400 ProblemDetails; DB timeout → 503 + retry-after; Empty result → 200 + empty array.

---

### 12.3 Lansweeper Scheduled Delta Sync

```
Scheduler (Celery Beat) → Worker: Trigger DeltaSyncJob(tenant_id)
Worker → Config/Secrets: Fetch Lansweeper credentials (Vault)
Worker → Lansweeper GraphQL: Query assets changed since cursor
Lansweeper → Worker: Asset changes (normalized)
Worker → Domain: SyncAssetsUseCase(tenant, changes)
Domain → OwnershipPolicy: For each field, determine writer
Domain → Repository: Upsert with conflict detection
Repository → PostgreSQL: Transactional upsert + conflict rows
Worker → Audit: Record sync event (success/conflict count)
Worker → Metrics: Increment sync.duration, sync.conflicts, sync.assets
Worker → Next Cursor: Persist for next delta run
```

**Failure Paths**: GraphQL error → Retry/backoff; Mapping error → Quarantine + alert; Conflict → Create `SyncConflict` record + notify; Partial failure → Transaction rollback per asset.

---

### 12.4 Lansweeper Webhook Processing

```
Lansweeper → Traefik: POST /webhooks/lansweeper (HMAC verified)
Traefik → Backend: Forward
Backend → Webhook Handler: Verify signature, extract tenant mapping
Backend → Dedupe: Check event_id processed (Redis SETNX)
Backend → Worker: Enqueue ProcessWebhookJob(tenant, payload)
Worker → Domain: ProcessAssetChange(tenant, change)
Domain → OwnershipPolicy: Apply field ownership
Domain → Repository: Upsert / Flag conflict
Worker → Audit: Record webhook processing
Worker → Metrics: webhook.latency, webhook.success, webhook.conflicts
```

**Failure Paths**: Invalid HMAC → 401 + audit; Duplicate event_id → 200 OK (idempotent); Tenant not mapped → 404 + alert; Processing error → Retry → DLQ.

---

### 12.5 Sync Failure & Retry Flow

```
Job Fails (Exception) → Celery: Retry (count < max_retries)
    │
    ├─ Transient (network, 5xx) → Exponential backoff + jitter → Re-queue
    │
    ├─ Auth Failure (401/403) → Alert Ops → Pause tenant sync → Manual resolve
    │
    ├─ Mapping/Schema Error → Quarantine batch → Alert → No partial commit
    │
    └─ Max Retries Exceeded → Dead Letter Queue (DLQ)
                                  │
                                  ├─ DLQ Consumer: Alert, enrich context
                                  ├─ Operator: Inspect, fix root cause, replay
                                  └─ Audit: Record DLQ entry + resolution
```

---

## 13. Current vs Target State Matrix

| Area | Current Evidence | Target State | Gap | Priority |
|---|---|---|---|---|
| **Code/Runtime** | `*.description.md` skeletons, directory structure | Executable backend, frontend, worker, mobile, infra | Implementation | 🔴 Critical |
| **OpenAPI Contract** | Descriptor/skeleton (`platform/api/contracts/openapi.yaml.description.md`) | Complete OpenAPI 3.1 + endpoint catalog + codegen | Spec completion + codegen | 🔴 Critical |
| **Tenancy** | Design intent (`09-multi-tenancy.md`) | Enforced checks + RLS policies + negative tests | Implementation + tests | 🔴 Critical |
| **Lansweeper Sync** | Design/prompt intent (`08-lansweeper-integration.md`) | Verified full/delta/bulk/webhook + idempotency + ownership matrix | Implementation + integration tests | 🔴 Critical |
| **Clean Architecture** | Dependency rules documented (`04-clean-architecture.md`) | Import-linter + arch tests in CI + ADR exceptions | Tooling + CI gates | 🟡 High |
| **BFF Pattern** | Responsibilities defined (`07-bff-pattern.md`) | Auth/CSRF/CORS tests + contract tests + tenant-negative tests | Implementation + tests | 🟡 High |
| **Event-Driven** | Candidate catalog (`10-event-driven.md`) | Broker config + retry tests + idempotency tests + runbooks | Implementation + runbooks | 🟡 High |
| **Data Model/RLS** | Requirements (`05-data-model.md`) | Approved ERD + migrations + RLS policies + index review | Schema + migrations | 🟡 High |
| **Observability** | Requirements in principles | OTel instrumentation + dashboards + alerts + log aggregation | Instrumentation | 🟡 High |
| **Security** | Threat model + controls matrix (`../07-security/`) | Pen test + dependency scan + secret scan + compliance evidence | Hardening + validation | 🟡 High |
| **CI/CD** | Jenkinsfile descriptor (`platform/ci/jenkins/Jenkinsfile.description.md`) | Pipeline gates: lint, typecheck, test, contract, security, arch | Pipeline implementation | 🟡 High |
| **Mobile** | Android skeleton (`mobile/android/README.md`) | Feature parity (auth, assets, offline sync) | Implementation | 🟢 Medium |
| **Documentation** | This doc + expanded design docs | Runbooks, API reference, operator guides, architecture decision log | Completion | 🟢 Medium |

---

## 14. Governance, ADRs & Compliance

### 14.1 Architecture Decision Records (ADRs)

| ADR | Title | Status | Location |
|---|---|---|---|
| 0001 | Clean Architecture Adoption | Accepted | `docs/12-adr/0001-clean-architecture.md` |
| 0002 | Multi-Tenancy: RLS + Application AuthZ | Accepted | `docs/12-adr/0002-multi-tenancy-rls.md` |
| 0003 | OpenAPI-First Contract Development | Accepted | `docs/12-adr/0003-openapi-first.md` |
| 0004 | BFF for Web, Direct API for Mobile | Accepted | `docs/12-adr/0004-bff-pattern.md` |
| 0005 | Lansweeper Field Ownership Matrix | Accepted | `docs/12-adr/0005-lansweeper-ownership.md` |
| 0006 | Idempotent Sync & Event Processing | Accepted | `docs/12-adr/0006-idempotent-sync.md` |
| 0007 | PostgreSQL 16 + Redis 7 Baseline | Accepted | `docs/12-adr/0007-data-stores.md` |
| 0008 | Traefik v3 Edge + Let's Encrypt | Accepted | `docs/12-adr/0008-edge-tls.md` |
| 0009 | Celery + Redis for Async Processing | Accepted | `docs/12-adr/0009-async-processing.md` |
| 0010 | Observability Stack: Prom/Grafana/Loki/OTel | Accepted | `docs/12-adr/0010-observability.md` |

> **Process** — New ADR required for: dependency rule exceptions, new external integrations, data store changes, auth/tenancy model changes, API breaking changes.

### 14.2 Compliance Targets

| Standard | Scope | Evidence Required |
|---|---|---|
| **SOC 2 Type II** | Security, Availability, Confidentiality | Audit report, control matrix, penetration test |
| **ISO 27001** | ISMS | Risk treatment, statement of applicability, internal audit |
| **GDPR** | Personal data processing | DPIA, records of processing, DPA, breach notification |
| **PCI DSS** (if billing) | Payment data | SAQ/RoC, ASV scan, penetration test |

---

## 15. Evidence & Verification Requirements

### 15.1 Definition of Done — Architecture Baseline

| Artifact | Verification Method | Status |
|---|---|---|
| **OpenAPI Spec Complete** | `spectral lint` + `openapi-generator` produces valid client/server | 🟡 Pending |
| **Import Linter Clean** | `import-linter` passes on all bounded contexts | 🟡 Pending |
| **Architecture Tests Pass** | Custom pytest suite: 0 dependency violations | 🟡 Pending |
| **RLS Policies Active** | `psql` test: cross-tenant query returns 0 rows | 🟡 Pending |
| **Tenant Negative Tests** | Pytest: 100% cross-tenant scenarios return 403/404 | 🟡 Pending |
| **Lansweeper Sync Verified** | Integration test: full/delta/bulk/webhook + conflict scenarios | 🟡 Pending |
| **Idempotency Proven** | Chaos test: duplicate delivery → no duplicate state | 🟡 Pending |
| **Contract Tests Pass** | Pact/Schemathesis: BFF↔Backend, Mobile↔API | 🟡 Pending |
| **CI Gates Enforced** | Pipeline fails on: lint, typecheck, test, contract, security, arch | 🟡 Pending |
| **Observability Live** | Dashboards show: RED metrics, traces, logs for all services | 🟡 Pending |
| **Runbooks Published** | `docs/11-operations/runbooks/*.md` for top 10 scenarios | 🟡 Pending |

### 15.2 Traceability Matrix (Sample)

| Requirement | Design Doc | ADR | Implementation | Test | Evidence |
|---|---|---|---|---|---|
| Clean Architecture | 04-clean-architecture.md | 0001 | `backend/nexusasset/*/domain` | import-linter | CI log |
| Multi-Tenancy RLS | 09-multi-tenancy.md | 0002 | Migrations + middleware | Cross-tenant pytest | Test report |
| Lansweeper Ownership | 08-lansweeper-integration.md | 0005 | `nxa_lansweeper/domain/policy` | Conflict scenario tests | Test report |
| OpenAPI-First | 06-api-contract.md | 0003 | `api/contracts/openapi.yaml` | Spectral + codegen | CI artifact |
| Idempotent Sync | 10-event-driven.md | 0006 | `nxa_lansweeper/application/sync` | Duplicate delivery test | Test report |

---

## Appendix A — Quick Reference: Canonical Naming

| Category | Convention | Examples |
|---|---|---|
| **Repository** | `nexusasset-cloud/` | — |
| **Django Package** | `backend/nexusasset/` | — |
| **Apps (Bounded Contexts)** | `nxa_<domain>` | `nxa_accounts`, `nxa_assets`, `nxa_lansweeper` |
| **Services** | `nxa-<role>` | `nxa-backend`, `nxa-frontend`, `nxa-worker`, `nxa-db`, `nxa-redis` |
| **Edge** | `nxa-traefik-public`, `nxa-le` | — |
| **Domains** | `app.nexusasset.cloud`, `api.nexusasset.cloud` | — |
| **NPM Packages** | `@nexusasset/<name>` | `@nexusasset/api-client`, `@nexusasset/ui-tokens` |
| **Secrets/Env** | `NXA_<SERVICE>_<KEY>` | `NXA_DB_PASSWORD`, `NXA_LANSWEeper_GRAPHQL_TOKEN` |
| **API Contract** | `api/contracts/openapi.yaml` | — |

---

## Appendix B — Status Legend

| Marker | Meaning |
|---|---|
| ✅ | **Verified** — Implemented, tested, evidence in repo/CI |
| 🟡 | **Descriptor/Target** — Designed, documented, not yet implemented |
| 🔴 | **Gap/Critical** — Missing, blocking, requires immediate action |
| 🟢 | **Planned/Medium** — Designed, lower priority |
| ○ | **Not Started** |
| ● | **In Progress** |
| ✗ | **Blocked/Failed** |

---

## Appendix C — Related Documents

| Document | Path |
|---|---|
| C4 Context & Boundaries | `docs/02-design/01-c4-context.md` |
| Container Topology | `docs/02-design/02-c4-container.md` |
| Component Details | `docs/02-design/03-c4-component.md` |
| Clean Architecture Rules | `docs/02-design/04-clean-architecture.md` |
| Data Model & RLS | `docs/02-design/05-data-model.md` |
| API Contract | `docs/02-design/06-api-contract.md` |
| BFF Pattern | `docs/02-design/07-bff-pattern.md` |
| Lansweeper Integration | `docs/02-design/08-lansweeper-integration.md` |
| Multi-Tenancy | `docs/02-design/09-multi-tenancy.md` |
| Event-Driven | `docs/02-design/10-event-driven.md` |
| Sequence Diagrams | `docs/02-design/11-sequence-diagrams.md` |
| Design Index | `docs/02-design/README.md` |
| ADR Index | `docs/12-adr/README.md` |
| Security Index | `docs/07-security/README.md` |
| Implementation Index | `docs/03-implementation/README.md` |

---

**End of file — NexusAsset Cloud / docs@nexusasset.cloud**