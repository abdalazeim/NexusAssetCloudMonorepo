# NexusAsset Cloud — Requirements Traceability Matrix (RTM)

## Status
Target planning artifact. This RTM defines the required traceability structure; entries are seeded from current documentation intent and must be verified/expanded during implementation.

## Traceability Columns
| Column | Description |
|---|---|
| Requirement ID | Stable ID (`FR-*`, `NFR-*`, `SEC-*`, `OPS-*`, `INT-*`). |
| Type | Functional, non-functional, security, operational, integration. |
| Requirement | Concise, testable requirement statement. |
| Priority | MVP / Post-MVP / Planned. |
| Bounded Context | `nxa_core/accounts/tenants/assets/lansweeper/billing/notifications/audit` or cross-cutting. |
| Contract/API | OpenAPI path/operation or BFF/mobile contract (Target/Unverified until implemented). |
| Implementation Evidence | Code, descriptor, config, migration, or "Unverified". |
| Test Evidence | Unit/integration/contract/E2E/security/load test or "Unverified". |
| Status | Implemented (Verified), Implemented (Descriptor Evidence), Target, Planned, Unverified. |

## Seed Matrix

| Requirement ID | Type | Requirement | Priority | Bounded Context | Contract/API | Implementation Evidence | Test Evidence | Status |
|---|---|---|---|---|---|---|---|---|
| FR-001 | Functional | System shall authenticate users and issue/validate access tokens according to approved JWT/BFF design. | MVP | `nxa_accounts` | Target: auth endpoints in OpenAPI 3.1 | Descriptor/prompt intent; no verified runtime implementation | Unverified | Target |
| FR-002 | Functional | System shall resolve tenant context and prevent cross-tenant data access. | MVP | `nxa_tenants`, cross-cutting | Target: tenant-aware APIs; never trust client tenant for authorization | Descriptor/prompt intent; RLS target | Unverified | Target |
| FR-003 | Functional | System shall manage asset inventory and relationships (CMDB/ITAM baseline). | MVP | `nxa_assets` | Target: assets CRUD/list/filter/detail | `platform/backend/apps/nxa_assets` structure/descriptors | Unverified | Target |
| FR-004 | Functional | System shall ingest Lansweeper discovery data through approved sync paths. | MVP/Post-MVP by path | `nxa_lansweeper`, `nxa_assets` | Target: integration/sync/webhook operations | `platform/backend/apps/nxa_lansweeper` descriptors; integration design docs | Unverified | Target |
| FR-005 | Functional | System shall record significant business/sync/security actions in audit events. | MVP | `nxa_audit` | Target: audit read/query operations | `nxa_audit` descriptors | Unverified | Target |
| FR-006 | Functional | System shall provide notifications for relevant operational/business events. | Post-MVP | `nxa_notifications` | Target | `nxa_notifications` descriptors | Unverified | Planned |
| FR-007 | Functional | System shall support billing/commercial records linked to governed business data. | Post-MVP | `nxa_billing` | Target | `nxa_billing` descriptors | Unverified | Planned |
| NFR-001 | Non-functional | Architecture shall enforce Clean Architecture dependency direction. | MVP | Cross-cutting | N/A | Design/ADR/prompt intent; no verified import-linter/CI evidence | Unverified | Target |
| NFR-002 | Non-functional | API contracts shall be OpenAPI 3.1 source of truth. | MVP | Cross-cutting | `platform/api/contracts/openapi.yaml.description.md` currently skeleton | Descriptor evidence only | Unverified | Implemented (Descriptor Evidence) / Target |
| NFR-003 | Non-functional | Lansweeper synchronization shall be idempotent, retry-safe, rate-limit aware, and observable. | MVP for selected paths | `nxa_lansweeper` | Target sync/webhook/status operations | Design/prompt evidence | Unverified | Target |
| SEC-001 | Security | Secrets shall use `NXA_*` naming and never be committed. | MVP | Cross-cutting | N/A | `.env.example.description.md`, governance/prompt evidence | Unverified | Target |
| SEC-002 | Security | Tenant isolation shall be enforced by application authorization and PostgreSQL RLS where applicable. | MVP | `nxa_tenants`, data layer | Target | Design intent; no verified RLS policies/migrations | Unverified | Target |
| OPS-001 | Operational | System shall expose health/readiness endpoints and structured operational telemetry. | MVP | Cross-cutting | Target `/healthz` and service health endpoints | URL/settings descriptors show intent; runtime unverified | Unverified | Target |
| OPS-002 | Operational | CI/CD shall enforce quality and security gates before promotion. | Post-MVP foundation / staged | Delivery | N/A | Jenkins descriptor and prompts | Unverified | Target |

## Required Next Actions
1. Replace/expand seed rows with approved product requirements from `docs/01-analysis/*`.
2. Assign owners and MVP scope decisions.
3. Link each MVP row to a specific OpenAPI operation ID/path and acceptance criteria.
4. Add evidence links as implementation progresses; never mark Verified without proof.
