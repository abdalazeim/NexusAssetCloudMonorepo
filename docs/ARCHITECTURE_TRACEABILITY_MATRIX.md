# ═══════════════════════════════════════════════════════════════
#  NEXUSASSET CLOUD — ARCHITECTURE TO IMPLEMENTATION TRACEABILITY
# ═══════════════════════════════════════════════════════════════
#  Document ID        : TR-001
#  Version            : 1.0.0
#  Status             : Baseline (Target/Design — Cross-Reference Matrix)
#  Classification     : Internal — Architecture & Engineering
#  Owner              : Architecture Guild / Platform Engineering
#  Last Generated     : 2026-11-09
#  Source Architecture: docs/ENTERPRISE_ARCHITECTURE.md (EA-001)
#  Target Repo        : nexusasset-cloud/
# ═══════════════════════════════════════════════════════════════

# NexusAsset Cloud — Architecture to Implementation Traceability Matrix

> **Purpose** — Map every architectural concept from `ENTERPRISE_ARCHITECTURE.md` (EA-001) to its corresponding implementation files, documentation, prompts, and configuration across the entire repository. This enables impact analysis, gap detection, and verification that the codebase reflects the target architecture.

---

## Table of Contents
1. [Architectural Principles → Files](#1-architectural-principles--files)
2. [System Context (C4 L1) → Files](#2-system-context-c4-l1--files)
3. [Container Topology (C4 L2) → Files](#3-container-topology-c4-l2--files)
4. [Component Architecture (C4 L3) → Files](#4-component-architecture-c4-l3--files)
5. [Clean Architecture & Dependency Rules → Files](#5-clean-architecture--dependency-rules--files)
6. [Data Model, ERD & RLS → Files](#6-data-model-erd--rls--files)
7. [API Contract & Standards → Files](#7-api-contract--standards--files)
8. [BFF Pattern → Files](#8-bff-pattern--files)
9. [Lansweeper Integration → Files](#9-lansweeper-integration--files)
10. [Multi-Tenancy Strategy → Files](#10-multi-tenancy-strategy--files)
11. [Event-Driven Architecture → Files](#11-event-driven-architecture--files)
12. [Key Sequence Flows → Files](#12-key-sequence-flows--files)
13. [Current vs Target State → Files](#13-current-vs-target-state--files)
14. [Governance, ADRs & Compliance → Files](#14-governance-adrs--compliance--files)
15. [Evidence & Verification → Files](#15-evidence--verification--files)
16. [Repository Structure Mapping](#16-repository-structure-mapping)

---

## 1. Architectural Principles → Files

| Principle (EA-001 §1) | Implementation Files | Documentation | Prompts | Config |
|---|---|---|---|---|
| **Clean Architecture** (Interfaces→App→Domain) | `platform/backend/apps/nxa_*/domain/`, `application/`, `infrastructure/`, `interfaces/` | `docs/02-design/04-clean-architecture.md`, `docs/03-implementation/04-clean-code.md`, `docs/03-implementation/05-dependency-rule.md` | `prompts/02-backend-clean-architecture.md` | `platform/backend/nexusasset/di/README.md` |
| **DDD Bounded Contexts** (`nxa_*`) | `platform/backend/apps/nxa_accounts/`, `nxa_tenants/`, `nxa_assets/`, `nxa_lansweeper/`, `nxa_billing/`, `nxa_notifications/`, `nxa_audit/`, `nxa_core/` | `docs/01-analysis/08-bounded-contexts.md`, `docs/04-domain-design/01-bounded-contexts.md` | `prompts/02-domain-analysis.md` | — |
| **OpenAPI-First** | `platform/api/contracts/openapi.yaml.description.md`, `platform/contracts/openapi/openapi.yaml` | `docs/02-design/06-api-contract.md`, `docs/09-api/01-openapi-spec.md` | `prompts/03-api-openapi-and-contracts.md` | `platform/packages/@nexusasset/api-client/` |
| **BFF for Web** | `platform/frontend/src/app/api/`, `platform/frontend/src/lib/api/`, `platform/frontend/proxy.ts` | `docs/02-design/07-bff-pattern.md`, `docs/09-api/README.md` | `prompts/05-frontend-bff.md` | `platform/frontend/next.config.ts` |
| **Multi-Tenancy by Design** | `platform/backend/apps/nxa_tenants/`, `shared/tenant_context/`, `platform/infrastructure/postgres/rls/` | `docs/02-design/09-multi-tenancy.md`, `docs/11-database/03-rls-policies.md` | `prompts/04-database-multitenancy.md` | `platform/backend/nexusasset/settings/base.py.description.md` |
| **Idempotent Integrations** | `platform/backend/apps/nxa_lansweeper/domain/sync_policy.py.description.md`, `application/sync_usecase.py.description.md` | `docs/02-design/08-lansweeper-integration.md`, `docs/17-integrations/01-lansweeper.md` | `prompts/04-lansweeper-integration.md` | — |
| **Observability & Auditability** | `platform/backend/apps/nxa_audit/`, `shared/observability/`, `platform/observability/` | `docs/06-operations/01-observability.md`, `docs/14-observability/README.md` | `prompts/11-observability-and-operations.md` | `platform/observability/otel/collector/config.yaml` |

---

## 2. System Context (C4 L1) → Files

| C4 Element (EA-001 §2) | Implementation Files | Documentation | Diagrams |
|---|---|---|---|
| **Actors: Web Users** | `platform/frontend/src/app/(auth)/`, `platform/frontend/src/features/auth/` | `docs/00-overview/01-vision.md`, `docs/19-user-guides/01-getting-started.md` | `docs/13-diagrams/system-context.txt` |
| **Actors: Mobile Users** | `platform/mobile/android/app/src/main/java/com/nexusasset/cloud/` | `docs/08-mobile/01-android-architecture.md`, `docs/08-mobile/03-api-integration.md` | — |
| **External: Lansweeper** | `platform/backend/apps/nxa_lansweeper/infrastructure/graphql_client.py.description.md`, `webhook_verifier.py.description.md` | `docs/02-design/08-lansweeper-integration.md`, `docs/17-integrations/01-lansweeper.md` | `docs/13-diagrams/data-flow.txt` |
| **External: Email/SMS/Push** | `platform/backend/apps/nxa_notifications/infrastructure/` | `docs/08-mobile/05-push-notifications.md` | — |
| **CI/CD Systems** | `platform/deployment/ci-cd/Jenkinsfile.description.md`, `platform/deployment/ci-cd/github/workflows/` | `docs/05-deployment/04-jenkins.md`, `docs/13-devops-cicd/01-jenkins.md` | `docs/13-diagrams/deployment-diagram.txt` |
| **Trust Boundaries** | `platform/infrastructure/traefik/`, `platform/security/access-control/`, `platform/security/tenant-isolation/` | `docs/07-security/01-threat-model.md`, `docs/07-security/05-authorization.md` | `docs/13-diagrams/network-topology.txt` |

---

## 3. Container Topology (C4 L2) → Files

| Container (EA-001 §3.1) | Implementation Files | Docker/Infra | Documentation |
|---|---|---|---|
| **nxa-frontend** (Next.js 15 + React 19) | `platform/frontend/src/`, `platform/frontend/package.json.description.md` | `platform/infrastructure/docker/frontend.Dockerfile`, `compose.yaml` | `docs/02-design/02-c4-container.md`, `docs/05-deployment/02-docker.md` |
| **nxa-backend** (Django 5 + DRF) | `platform/backend/`, `platform/backend/manage.py.description.md`, `platform/backend/pyproject.toml.description.md` | `platform/infrastructure/docker/backend.Dockerfile`, `compose.yaml` | `docs/02-design/02-c4-container.md`, `docs/03-implementation/01-repository-structure.md` |
| **nxa-worker** (Celery) | `platform/backend/apps/*/interfaces/tasks/`, `platform/worker/` | `platform/infrastructure/docker/worker.Dockerfile`, `platform/infrastructure/docker/scheduler.Dockerfile` | `docs/02-design/10-event-driven.md`, `docs/05-deployment/08-redis-celery.md` |
| **nxa-db** (PostgreSQL 16 + RLS) | `platform/infrastructure/postgres/`, `platform/backend/apps/*/migrations/` | `platform/infrastructure/docker/`, `compose.yaml` (postgres service) | `docs/11-database/README.md`, `docs/05-deployment/07-postgresql.md` |
| **nxa-redis** (Redis 7) | `platform/infrastructure/redis/config/`, `platform/backend/nexusasset/celery.py.description.md` | `compose.yaml` (redis service) | `docs/05-deployment/08-redis-celery.md` |
| **nxa-traefik-public** (Traefik v3) | `platform/infrastructure/traefik/traefik.yml`, `dynamic/` | `platform/infrastructure/docker/`, `compose.yaml` (traefik service) | `docs/05-deployment/03-traefik.md`, `docs/13-diagrams/network-topology.txt` |
| **nxa-le** (Cert-manager/LE) | `platform/infrastructure/traefik/tls/`, `dynamic/tls.yml` | `compose.yaml` | `docs/05-deployment/03-traefik.md` |

### Runtime Concerns (EA-001 §3.2) → Files

| Concern | Implementation Files | Documentation |
|---|---|---|
| Health/Readiness | `platform/frontend/src/app/api/health/`, `platform/backend/nexusasset/urls.py.description.md` | `docs/06-operations/01-observability.md` |
| Migrations | `platform/backend/apps/*/migrations/`, `platform/infrastructure/postgres/migrations/` | `docs/11-database/02-migrations.md`, `docs/03-implementation/09-migrations-strategy.md` |
| Worker Scaling | `platform/worker/queues/`, `platform/deployment/ansible/roles/application/` | `docs/16-performance-scalability/04-load-testing.md` |
| Secret Injection | `platform/security/secrets-management/sops/`, `.env.example` | `docs/07-security/03-secrets-management.md` |
| Tenant-Safe Config | `platform/backend/shared/tenant_context/`, `platform/backend/nexusasset/settings/` | `docs/02-design/09-multi-tenancy.md` |
| Logging/Metrics/Traces | `platform/observability/otel/`, `platform/backend/shared/observability/` | `docs/14-observability/README.md` |
| Backup/Restore | `platform/ops/backup-restore/`, `platform/infrastructure/postgres/backup/` | `docs/15-disaster-recovery/01-dr-plan.md`, `docs/06-operations/09-backup-dr.md` |

---

## 4. Component Architecture (C4 L3) → Files

| Bounded Context (EA-001 §4.2) | Domain | Application | Infrastructure | Interfaces | Tests |
|---|---|---|---|---|---|
| **nxa_accounts** | `platform/backend/apps/nxa_accounts/domain/__init__.description.md` | `application/__init__.description.md` | `infrastructure/__init__.description.md` | `interfaces/__init__.description.md` | `tests/unit/`, `integration/`, `api/` |
| **nxa_tenants** | `platform/backend/apps/nxa_tenants/domain/__init__.description.md` | `application/__init__.description.md` | `infrastructure/__init__.description.md` | `interfaces/__init__.description.md` | `tests/unit/`, `integration/`, `api/` |
| **nxa_assets** | `platform/backend/apps/nxa_assets/domain/__init__.description.md` | `application/__init__.description.md` | `infrastructure/__init__.description.md` | `interfaces/__init__.description.md` | `tests/unit/`, `integration/`, `api/` |
| **nxa_lansweeper** | `platform/backend/apps/nxa_lansweeper/domain/__init__.description.md` | `application/__init__.description.md` | `infrastructure/__init__.description.md` | `interfaces/__init__.description.md` | `tests/unit/`, `integration/`, `api/`, `contract/` |
| **nxa_billing** | `platform/backend/apps/nxa_billing/domain/__init__.description.md` | `application/__init__.description.md` | `infrastructure/__init__.description.md` | `interfaces/__init__.description.md` | `tests/unit/`, `integration/`, `api/` |
| **nxa_notifications** | `platform/backend/apps/nxa_notifications/domain/__init__.description.md` | `application/__init__.description.md` | `infrastructure/__init__.description.md` | `interfaces/__init__.description.md` | `tests/unit/`, `integration/`, `api/` |
| **nxa_audit** | `platform/backend/apps/nxa_audit/domain/__init__.description.md` | `application/__init__.description.md` | `infrastructure/__init__.description.md` | `interfaces/__init__.description.md` | `tests/unit/`, `integration/`, `api/` |
| **nxa_core** | `platform/backend/apps/nxa_core/domain/__init__.description.md` | `application/__init__.description.md` | `infrastructure/__init__.description.md` | `interfaces/__init__.description.md` | `tests/unit/`, `integration/`, `api/` |

### Frontend Components (EA-001 §4.3) → Files

| Area | Implementation Files |
|---|---|
| **App Shell** | `platform/frontend/src/app/layout.tsx`, `page.tsx`, `globals.css`, `providers/` |
| **Auth/Session** | `platform/frontend/src/app/(auth)/`, `features/auth/`, `lib/auth/` |
| **BFF Handlers** | `platform/frontend/src/app/api/`, `proxy.ts` |
| **API Client** | `platform/frontend/src/lib/api/`, `platform/packages/@nexusasset/api-client/` |
| **Feature Modules** | `platform/frontend/src/features/assets/`, `tenants/`, `lansweeper/`, `audit/`, `billing/` |
| **Shared UI** | `platform/frontend/src/components/ui/`, `layout/`, `forms/`, `tables/`, `charts/`, `feedback/`, `navigation/` |

---

## 5. Clean Architecture & Dependency Rules → Files

| Rule (EA-001 §5) | Enforcement Files | Documentation | CI/CD |
|---|---|---|---|
| **Domain: No framework imports** | `platform/backend/apps/*/domain/` (verified by import-linter) | `docs/02-design/04-clean-architecture.md`, `docs/03-implementation/05-dependency-rule.md` | `platform/tests/architecture/import_linter.yml`, `platform/quality/lint/python/` |
| **Application: No concrete ORM** | `platform/backend/apps/*/application/` | `docs/03-implementation/06-use-case-pattern.md` | `platform/tests/architecture/layer_tests.py` |
| **Infrastructure: Implements ports** | `platform/backend/apps/*/infrastructure/` | `docs/03-implementation/07-repository-pattern.md`, `docs/03-implementation/08-mappers.md` | — |
| **Interfaces: Translates HTTP→App** | `platform/backend/apps/*/interfaces/` | `docs/03-implementation/10-error-handling.md` | `platform/tests/architecture/boundary_tests.py` |
| **Dependency Direction** | `platform/backend/nexusasset/di/README.md` (composition root) | `docs/03-implementation/01-repository-structure.md` §6 | `platform/quality-gate.sh` |

---

## 6. Data Model, ERD & RLS → Files

| Requirement (EA-001 §6) | Implementation Files | Documentation | Migrations |
|---|---|---|---|
| **Tenant Scope** (`tenant_id` + RLS) | `platform/backend/apps/*/domain/` entities, `platform/infrastructure/postgres/rls/` | `docs/02-design/05-data-model.md`, `docs/11-database/01-data-model.md`, `docs/11-database/03-rls-policies.md` | `platform/backend/apps/*/migrations/`, `platform/infrastructure/postgres/migrations/` |
| **Identity/Integrity** (PK, FK, constraints) | `platform/backend/apps/*/infrastructure/` (ORM models) | `docs/03-implementation/09-migrations-strategy.md` | `platform/backend/apps/*/migrations/` |
| **Provenance** (source_id, source_system) | `platform/backend/apps/nxa_lansweeper/domain/discovery.py.description.md`, `nxa_assets/domain/` | `docs/02-design/08-lansweeper-integration.md` §Field Ownership | — |
| **Auditability** | `platform/backend/apps/nxa_audit/domain/`, `application/` | `docs/02-design/03-c4-component.md`, `docs/07-security/README.md` | `platform/backend/apps/nxa_audit/migrations/` |
| **Performance Indexes** | `platform/infrastructure/postgres/migrations/`, `platform/backend/apps/*/migrations/` | `docs/11-database/04-indexing.md`, `docs/16-performance-scalability/03-database-scaling.md` | — |
| **Retention** | `platform/ops/database-operations/`, `platform/backend/apps/nxa_audit/application/` | `docs/11-database/05-retention.md`, `docs/15-disaster-recovery/01-dr-plan.md` | — |

---

## 7. API Contract & Standards → Files

| Standard (EA-001 §7) | Implementation Files | Documentation | Codegen |
|---|---|---|---|
| **OpenAPI 3.1 Source of Truth** | `platform/contracts/openapi/openapi.yaml`, `platform/api/contracts/openapi.yaml.description.md` | `docs/02-design/06-api-contract.md`, `docs/09-api/01-openapi-spec.md` | `platform/contracts/generated/typescript/`, `kotlin/`, `swift/` |
| **Versioning** | `platform/contracts/openapi/versions/v1/`, API routes | `docs/09-api/03-versioning.md` | — |
| **Naming Conventions** | `platform/contracts/openapi/paths/*.yaml` | `docs/03-implementation/02-naming-conventions.md`, `docs/09-api/02-authentication.md` | — |
| **Operation IDs** | `platform/contracts/openapi/paths/*.yaml` | `docs/09-api/01-openapi-spec.md` | `platform/packages/@nexusasset/api-client/src/` |
| **Auth Requirements** | `platform/contracts/openapi/schemas/common.yaml` (securitySchemes) | `docs/09-api/06-authentication.md`, `docs/07-security/04-authentication.md` | — |
| **Pagination/Filter/Sort** | `platform/contracts/openapi/schemas/common.yaml` (Pagination, Filter) | `docs/09-api/04-pagination.md` | — |
| **Error Envelope** (RFC 9457) | `platform/contracts/openapi/schemas/common.yaml` (ProblemDetails) | `docs/09-api/05-error-codes.md`, `docs/03-implementation/10-error-handling.md` | — |
| **Idempotency** | `platform/contracts/openapi/paths/lansweeper.yaml`, `billing.yaml` | `docs/09-api/07-rate-limiting.md` | — |
| **Deprecation** | `platform/contracts/openapi/versions/` | `docs/09-api/03-versioning.md` | — |

---

## 8. BFF Pattern → Files

| Responsibility (EA-001 §8) | BFF Files | Backend Files | Tests |
|---|---|---|---|
| **UI Aggregation** | `platform/frontend/src/app/api/proxy/`, `features/*/api.ts` | `platform/backend/apps/*/interfaces/api/` | `platform/tests/integration/backend-frontend/` |
| **Browser Auth/Session** | `platform/frontend/src/lib/auth/`, `app/(auth)/`, `middleware.ts` | `platform/backend/apps/nxa_accounts/interfaces/`, `shared/tenant_context/` | `platform/tests/security/authentication/`, `tenant-isolation/` |
| **Business Rules** | ❌ NOT in BFF | ✅ `platform/backend/apps/*/application/`, `domain/` | `platform/tests/architecture/` |
| **Tenant Authorization** | `platform/frontend/src/lib/permissions/` | `platform/backend/apps/nxa_tenants/application/`, `shared/tenant_context/` | `platform/tests/security/tenant-isolation/` |
| **Contract Alignment** | `platform/frontend/src/lib/api/` (generated) | `platform/contracts/openapi/openapi.yaml` | `platform/tests/contract/` |

### Risks & Mitigations (EA-001 §8.2) → Files

| Risk | Mitigation Files |
|---|---|
| Business logic duplication | `platform/tests/architecture/` boundary tests, PR checklist `project-meta/templates/` |
| Token leakage | `platform/frontend/src/lib/auth/` (HttpOnly cookies), `platform/security/access-control/` |
| Bypassing AuthZ | `platform/tests/security/tenant-isolation/` negative tests |
| Contract divergence | `platform/tests/contract/`, `platform/scripts/generate-contracts.sh` |

---

## 9. Lansweeper Integration → Files

| Component (EA-001 §9) | Implementation Files | Documentation |
|---|---|---|
| **Sync Modes** (Full/Delta/Bulk/Webhook) | `platform/backend/apps/nxa_lansweeper/application/sync_usecase.py.description.md`, `interfaces/tasks/` | `docs/02-design/08-lansweeper-integration.md` §9.2, `docs/17-integrations/01-lansweeper.md` |
| **Field Ownership Matrix** | `platform/backend/apps/nxa_lansweeper/domain/sync_policy.py.description.md` | `docs/02-design/08-lansweeper-integration.md` §9.3, `docs/12-adr/0005-lansweeper-ownership.md` |
| **Idempotency & Conflict Resolution** | `platform/backend/apps/nxa_lansweeper/application/conflict_service.py.description.md`, `domain/events.py.description.md` | `docs/02-design/08-lansweeper-integration.md` §9.4 |
| **Failure Handling** | `platform/backend/apps/nxa_lansweeper/infrastructure/job_store.py.description.md`, `webhook_verifier.py.description.md` | `docs/02-design/08-lansweeper-integration.md` §9.5 |
| **GraphQL Client** | `platform/backend/apps/nxa_lansweeper/infrastructure/graphql_client.py.description.md` | `docs/17-integrations/01-lansweeper.md` |
| **Webhook Endpoint** | `platform/backend/apps/nxa_lansweeper/interfaces/webhooks/` | `docs/09-api/07-webhooks.md` |
| **Audit Trail** | `platform/backend/apps/nxa_audit/application/`, `nxa_lansweeper/application/sync_usecase.py.description.md` | `docs/02-design/08-lansweeper-integration.md` |

---

## 10. Multi-Tenancy Strategy → Files

| Layer (EA-001 §10) | Implementation Files | Documentation | Tests |
|---|---|---|---|
| **Application: Tenant Context** | `platform/backend/shared/tenant_context/`, `platform/backend/apps/nxa_tenants/application/` | `docs/02-design/09-multi-tenancy.md` §10.1 | `platform/tests/unit/` (tenant context) |
| **Data: RLS Policies** | `platform/infrastructure/postgres/rls/`, `platform/backend/apps/*/infrastructure/` (RLS-aware repos) | `docs/11-database/03-rls-policies.md`, `docs/12-adr/0008-multi-tenancy-strategy.md` | `platform/tests/security/tenant-isolation/` |
| **Infrastructure: Connection Pools** | `platform/backend/nexusasset/settings/base.py.description.md` (pool config) | `docs/02-design/09-multi-tenancy.md` §10.2 | — |
| **Observability: Tenant Tags** | `platform/backend/shared/observability/`, `platform/observability/otel/` | `docs/14-observability/02-logging.md`, `03-tracing.md` | — |
| **Admin/Cross-Tenant** | `platform/backend/apps/nxa_tenants/interfaces/admin/`, `shared/tenant_context/` (bypass) | `docs/02-design/09-multi-tenancy.md` §10.3 | `platform/tests/security/tenant-isolation/` (admin scenarios) |

---

## 11. Event-Driven Architecture → Files

| Requirement (EA-001 §11) | Implementation Files | Documentation |
|---|---|---|
| **Tenant Context in Events** | `platform/backend/shared/events/`, `platform/backend/apps/*/domain/events.py.description.md` | `docs/02-design/10-event-driven.md` §11.1 |
| **Idempotent Handlers** | `platform/backend/apps/*/application/` (event handlers), `platform/worker/tasks/` | `docs/02-design/10-event-driven.md` §11.1 |
| **Schema Versioning** | `platform/contracts/events/*.yaml`, `platform/contracts/asyncapi/asyncapi.yaml` | `docs/09-api/03-versioning.md` |
| **Observability** | `platform/backend/shared/observability/`, `platform/observability/otel/` | `docs/14-observability/README.md` |
| **Failure Handling** (Retry/DLQ) | `platform/worker/queues/`, `monitoring/`, `platform/backend/apps/*/application/` (retry logic) | `docs/02-design/10-event-driven.md` §11.1 |
| **Audit Emission** | `platform/backend/apps/nxa_audit/application/`, `platform/backend/shared/events/` | `docs/02-design/03-c4-component.md` |

### Event Catalog (EA-001 §11.2) → Files

| Event | Publisher | Implementation Files |
|---|---|---|
| `asset.discovered` | `nxa-lansweeper` | `platform/backend/apps/nxa_lansweeper/domain/events.py.description.md`, `nxa_assets/application/` |
| `asset.synced` | `nxa-lansweeper` | `platform/backend/apps/nxa_lansweeper/application/sync_usecase.py.description.md` |
| `sync.conflict.detected` | `nxa-lansweeper` | `platform/backend/apps/nxa_lansweeper/application/conflict_service.py.description.md` |
| `audit.recorded` | `nxa-audit` | `platform/backend/apps/nxa_audit/application/` |
| `notification.requested` | Any | `platform/backend/apps/nxa_notifications/application/` |
| `billing.entitlement.changed` | `nxa-billing` | `platform/backend/apps/nxa_billing/application/` |

---

## 12. Key Sequence Flows → Files

| Flow (EA-001 §12) | Implementation Files | Documentation | Tests |
|---|---|---|---|
| **User Login** | `platform/frontend/src/app/(auth)/login/`, `lib/auth/`, `platform/backend/apps/nxa_accounts/interfaces/`, `shared/tenant_context/` | `docs/13-diagrams/sequence-auth.txt`, `docs/02-design/11-sequence-diagrams.md` | `platform/tests/e2e/auth/`, `platform/tests/security/authentication/` |
| **Asset List/Detail** | `platform/frontend/src/features/assets/`, `lib/api/`, `platform/backend/apps/nxa_assets/interfaces/`, `application/`, `infrastructure/` | `docs/13-diagrams/sequence-sync.txt`, `docs/02-design/11-sequence-diagrams.md` | `platform/tests/api/assets/`, `platform/tests/integration/backend-frontend/` |
| **Lansweeper Delta Sync** | `platform/worker/tasks/`, `platform/backend/apps/nxa_lansweeper/application/sync_usecase.py.description.md`, `infrastructure/graphql_client.py.description.md` | `docs/02-design/11-sequence-diagrams.md` §12.3 | `platform/tests/integration/lansweeper/`, `platform/tests/contract/` |
| **Lansweeper Webhook** | `platform/backend/apps/nxa_lansweeper/interfaces/webhooks/`, `application/`, `worker/tasks/` | `docs/02-design/11-sequence-diagrams.md` §12.4 | `platform/tests/api/lansweeper/`, `platform/tests/security/` |
| **Sync Failure/Retry** | `platform/worker/tasks/`, `monitoring/`, `platform/backend/apps/nxa_lansweeper/infrastructure/job_store.py.description.md` | `docs/02-design/11-sequence-diagrams.md` §12.5 | `platform/tests/resilience/`, `platform/tests/load/` |

---

## 13. Current vs Target State → Files

| Area (EA-001 §13) | Current Evidence Files | Target Implementation Files | Gap Tracking |
|---|---|---|---|
| **Code/Runtime** | `platform/**/*.description.md` (skeletons) | `platform/backend/`, `frontend/`, `mobile/`, `worker/` (executable) | `docs/documentation-audit/FILE_ASSESSMENT_MATRIX.md` |
| **OpenAPI Contract** | `platform/api/contracts/openapi.yaml.description.md` (skeleton) | `platform/contracts/openapi/openapi.yaml` (complete) | `docs/documentation-audit/REQUIREMENTS_TRACEABILITY_MATRIX.md` |
| **Tenancy** | `docs/02-design/09-multi-tenancy.md` (design) | `platform/backend/apps/nxa_tenants/`, `shared/tenant_context/`, `postgres/rls/` | `docs/12-adr/0008-multi-tenancy-strategy.md` |
| **Lansweeper Sync** | `docs/02-design/08-lansweeper-integration.md` (design) | `platform/backend/apps/nxa_lansweeper/` (full impl) | `docs/12-adr/0005-lansweeper-ownership.md` |
| **Clean Architecture** | `docs/02-design/04-clean-architecture.md` (rules) | `platform/tests/architecture/`, `import-linter` in CI | `docs/03-implementation/05-dependency-rule.md` |
| **BFF Pattern** | `docs/02-design/07-bff-pattern.md` (responsibilities) | `platform/frontend/src/app/api/`, `lib/api/`, contract tests | `docs/09-api/08-codegen.md` |
| **Event-Driven** | `docs/02-design/10-event-driven.md` (catalog) | `platform/worker/`, `shared/events/`, `contracts/events/` | `docs/12-adr/0009-event-driven-architecture.md` |
| **Data Model/RLS** | `docs/02-design/05-data-model.md` (requirements) | `platform/infrastructure/postgres/rls/`, `migrations/` | `docs/11-database/03-rls-policies.md` |
| **Observability** | Principles in EA-001 §1 | `platform/observability/`, `shared/observability/`, OTel | `docs/14-observability/README.md` |
| **Security** | `docs/07-security/` (threat model, controls) | `platform/security/`, scans in CI, compliance evidence | `docs/07-security/SECURITY_CONTROLS_MATRIX.md` |
| **CI/CD** | `platform/infrastructure/jenkins/Jenkinsfile.description.md` | `platform/deployment/ci-cd/`, quality gates | `docs/13-devops-cicd/03-quality-gates.md` |
| **Mobile** | `platform/mobile/android/` (skeleton) | `platform/mobile/android/app/src/main/` (feature complete) | `docs/08-mobile/README.md` |
| **Documentation** | This matrix + `docs/` | Runbooks, API reference, operator guides | `docs/documentation-audit/` |

---

## 14. Governance, ADRs & Compliance → Files

| ADR (EA-001 §14.1) | ADR File | Related Implementation | Status |
|---|---|---|---|
| 0001 Clean Architecture | `docs/12-adr/0001-use-clean-architecture.md` | `platform/backend/apps/*/`, `platform/tests/architecture/` | ✅ Accepted |
| 0002 Multi-Tenancy RLS | `docs/12-adr/0002-multi-tenancy-rls.md` | `platform/infrastructure/postgres/rls/`, `shared/tenant_context/` | ✅ Accepted |
| 0003 OpenAPI-First | `docs/12-adr/0003-openapi-first.md` | `platform/contracts/openapi/`, `platform/scripts/generate-contracts.sh` | ✅ Accepted |
| 0004 BFF Pattern | `docs/12-adr/0004-bff-pattern.md` | `platform/frontend/src/app/api/`, `proxy.ts` | ✅ Accepted |
| 0005 Lansweeper Ownership | `docs/12-adr/0005-lansweeper-ownership.md` | `platform/backend/apps/nxa_lansweeper/domain/sync_policy.py.description.md` | ✅ Accepted |
| 0006 Idempotent Sync | `docs/12-adr/0006-idempotent-sync.md` | `platform/backend/apps/nxa_lansweeper/application/`, `worker/tasks/` | ✅ Accepted |
| 0007 Data Stores | `docs/12-adr/0007-data-stores.md` | `platform/infrastructure/postgres/`, `redis/`, `compose.yaml` | ✅ Accepted |
| 0008 Edge TLS | `docs/12-adr/0008-edge-tls.md` | `platform/infrastructure/traefik/`, `compose.yaml` | ✅ Accepted |
| 0009 Async Processing | `docs/12-adr/0009-async-processing.md` | `platform/worker/`, `shared/events/` | ✅ Accepted |
| 0010 Observability | `docs/12-adr/0010-observability.md` | `platform/observability/`, `shared/observability/` | ✅ Accepted |

### Compliance Targets (EA-001 §14.2) → Files

| Standard | Evidence Files | Platform Files |
|---|---|---|
| **SOC 2 Type II** | `platform/security/compliance/soc2/`, `platform/security/incident-response/` | `platform/security/policies/`, `platform/ops/incident-management/` |
| **ISO 27001** | `platform/security/compliance/iso27001/`, `project-meta/policies/` | `platform/security/iam/`, `access-control/` |
| **GDPR** | `platform/security/compliance/gdpr/`, `project-meta/policies/privacy-policy.md` | `platform/security/encryption/`, `tenant-isolation/` |
| **PCI DSS** | `platform/security/compliance/pci-dss/` (if billing) | `platform/security/encryption/`, `secrets-management/` |

---

## 15. Evidence & Verification → Files

| Definition of Done (EA-001 §15.1) | Verification Files | CI/CD Integration |
|---|---|---|
| **OpenAPI Spec Complete** | `platform/scripts/generate-contracts.sh`, `platform/tests/contract/` | `platform/quality-gate.sh`, GitHub Actions/Jenkins |
| **Import Linter Clean** | `platform/tests/architecture/import_linter.yml` | `platform/quality/lint/python/`, Jenkins quality gate |
| **Architecture Tests Pass** | `platform/tests/architecture/boundary_tests.py`, `layer_tests.py` | `platform/quality-gate.sh` |
| **RLS Policies Active** | `platform/infrastructure/postgres/rls/`, `platform/tests/security/tenant-isolation/` | `platform/tests/security/tenant-isolation/` in CI |
| **Tenant Negative Tests** | `platform/tests/security/tenant-isolation/` (cross-tenant → 403/404) | Jenkins/GitHub Actions security stage |
| **Lansweeper Sync Verified** | `platform/tests/integration/lansweeper/`, `contract/` | `platform/quality-gate.sh` integration stage |
| **Idempotency Proven** | `platform/tests/resilience/` (duplicate delivery tests) | `platform/tests/resilience/` in CI |
| **Contract Tests Pass** | `platform/tests/contract/provider/`, `consumer/` (Pact/Schemathesis) | `platform/quality-gate.sh` contract stage |
| **CI Gates Enforced** | `platform/deployment/ci-cd/quality-gates/`, `Jenkinsfile.description.md` | Jenkins pipeline + GitHub Actions |
| **Observability Live** | `platform/observability/` dashboards, `platform/observability/otel/` | `platform/deployment/ansible/roles/monitoring/` |
| **Runbooks Published** | `platform/ops/runbooks/`, `docs/06-operations/07-runbooks.md` | — |

### Traceability Matrix Sample (EA-001 §15.2) → Files

| Requirement | Design Doc | ADR | Implementation | Test | Evidence |
|---|---|---|---|---|---|
| Clean Architecture | `docs/02-design/04-clean-architecture.md` | 0001 | `platform/backend/apps/*/domain/` | `platform/tests/architecture/` | CI import-linter log |
| Multi-Tenancy RLS | `docs/02-design/09-multi-tenancy.md` | 0002 | `platform/infrastructure/postgres/rls/` | `platform/tests/security/tenant-isolation/` | Test report |
| Lansweeper Ownership | `docs/02-design/08-lansweeper-integration.md` | 0005 | `platform/backend/apps/nxa_lansweeper/domain/sync_policy.py.description.md` | `platform/tests/integration/lansweeper/` | Test report |
| OpenAPI-First | `docs/02-design/06-api-contract.md` | 0003 | `platform/contracts/openapi/openapi.yaml` | `platform/tests/contract/` | Spectral + codegen artifacts |
| Idempotent Sync | `docs/02-design/10-event-driven.md` | 0006 | `platform/backend/apps/nxa_lansweeper/application/sync_usecase.py.description.md` | `platform/tests/resilience/` | Duplicate delivery test report |

---

## 16. Repository Structure Mapping

| EA-001 Target Path | Actual Repository Path | Status |
|---|---|---|
| `nexusasset-cloud/` | `C:\Users\user176\Downloads\NexusAsset CloudMonorepo\` | ✅ Root |
| `docs/` (00–22) | `docs/00-overview/` ... `docs/22-appendices/` | ✅ Complete |
| `prompts/` (00–20) | `prompts/00-global-context-and-rules.md` ... `prompts/13-final-verification-and-release.md` | ✅ Complete (14 phases) |
| `platform/backend/` | `platform/backend/` | ✅ Skeleton (`.description.md`) |
| `platform/frontend/` | `platform/frontend/` | ✅ Skeleton (`.description.md`) |
| `platform/mobile/` | `platform/mobile/android/` | ✅ Skeleton |
| `platform/packages/@nexusasset/` | `platform/packages/@nexusasset/` | ✅ 6 packages |
| `platform/contracts/` | `platform/contracts/`, `platform/api/contracts/` | ✅ Descriptor exists |
| `platform/design-system/` | `platform/design-system/` | 🟡 Docs only (`docs/10-design-system/`) |
| `platform/infrastructure/` | `platform/infrastructure/` | ✅ Docker, Traefik, Ansible, Jenkins, monitoring |
| `platform/deployment/` | `platform/deployment/` | 🟡 Partial (ansible, ci-cd) |
| `platform/security/` | `platform/security/` | 🟡 Docs only (`docs/07-security/`) |
| `platform/observability/` | `platform/observability/` | 🟡 Docs only (`docs/06-operations/`, `docs/14-observability/`) |
| `platform/ops/` | `platform/ops/` | 🟡 Docs only (`docs/06-operations/`, `docs/08-operations/`) |
| `platform/tests/` | `platform/tests/` | ✅ Skeleton (`.description.md`) |
| `platform/quality/` | `platform/quality/` | 🟡 Partial |
| `platform/scripts/` | `platform/scripts/` | ✅ README only |
| `project-meta/` | `project-meta/` | ✅ Governance, policies, ADR templates |
| `archive/` | `archive/` | ✅ 7 traceability files |

---

## Appendix — Quick Navigation Index

| Architecture Section | Primary Doc | Key Implementation Root | Key Test Root |
|---|---|---|---|
| Principles | EA-001 §1 | `platform/backend/apps/` | `platform/tests/architecture/` |
| C4 Context | EA-001 §2 | `platform/infrastructure/traefik/` | `platform/tests/security/` |
| C4 Containers | EA-001 §3 | `platform/infrastructure/docker/` | `platform/tests/integration/` |
| C4 Components | EA-001 §4 | `platform/backend/apps/*/domain/` | `platform/tests/unit/` |
| Clean Architecture | EA-001 §5 | `platform/backend/nexusasset/di/` | `platform/tests/architecture/` |
| Data Model/RLS | EA-001 §6 | `platform/infrastructure/postgres/rls/` | `platform/tests/security/tenant-isolation/` |
| API Contracts | EA-001 §7 | `platform/contracts/openapi/` | `platform/tests/contract/` |
| BFF | EA-001 §8 | `platform/frontend/src/app/api/` | `platform/tests/integration/backend-frontend/` |
| Lansweeper | EA-001 §9 | `platform/backend/apps/nxa_lansweeper/` | `platform/tests/integration/lansweeper/` |
| Multi-Tenancy | EA-001 §10 | `platform/backend/shared/tenant_context/` | `platform/tests/security/tenant-isolation/` |
| Event-Driven | EA-001 §11 | `platform/worker/`, `shared/events/` | `platform/tests/resilience/` |
| Sequences | EA-001 §12 | (distributed) | `platform/tests/e2e/` |
| Current vs Target | EA-001 §13 | `docs/documentation-audit/` | — |
| ADRs | EA-001 §14 | `docs/12-adr/` | — |
| Verification | EA-001 §15 | `platform/quality-gate.sh` | All test directories |

---

**End of file — NexusAsset Cloud / docs@nexusasset.cloud**