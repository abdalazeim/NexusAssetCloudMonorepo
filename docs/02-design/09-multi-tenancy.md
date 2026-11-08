# NexusAsset Cloud — Multi-Tenancy Design

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (target design) |
| Evidence level | Design/prompt evidence; RLS policies and runtime enforcement Unverified |
| Primary evidence | `docs/02-design/09-multi-tenancy.md` (prior baseline), `docs/07-security/05-authorization.md`, `prompts/10-security-and-compliance.md`, `docs/01-analysis/08-bounded-contexts.md` |

## Purpose
Define how NexusAsset Cloud isolates tenant data and operations across API, application, background jobs, integrations, and PostgreSQL.

## Tenant Isolation Strategy (Target)
NexusAsset Cloud uses a **logical multi-tenancy** model: shared platform/services with strict tenant scoping. Tenant-owned records include `tenant_id` (or equivalent enforced scope), application logic enforces authorization, and PostgreSQL RLS provides defense-in-depth where applicable.

## Tenant Context Lifecycle (Target)
| Stage | Requirement | Status |
|---|---|---|
| Authentication | Identify principal and allowed tenant memberships/claims source | Target/Unverified |
| Tenant selection/resolution | Resolve effective tenant from trusted server-side rules (not arbitrary client override for authorization) | Target/Unverified |
| Request propagation | Pass tenant context to BFF/API/use cases/data access explicitly | Target/Unverified |
| Authorization | Verify principal can act on selected tenant/resource | Target/Unverified |
| Data access | Apply tenant filters and/or DB session settings needed for RLS | Target/Unverified |
| Async jobs | Store/restore validated tenant context; no global cross-tenant processing unless explicitly designed/admin-controlled | Target/Unverified |
| Integrations/webhooks | Map external events to tenant using verified configuration; reject ambiguous mappings | Target/Unverified |
| Audit | Record tenant-scoped sensitive actions | Target/Unverified |

## Data Classification
| Data type | Isolation rule |
|---|---|
| Tenant-owned business data (assets, billing records, tenant configs, tenant users/memberships as designed) | Must be tenant-scoped and protected by app + RLS target |
| Global/reference data (if any) | Explicitly classified, read/write controlled, no tenant leakage through joins or caches |
| Operational/control-plane data | Restricted administrative access, audited |

## PostgreSQL RLS (Target)
| Requirement | Detail | Status |
|---|---|---|
| Policy design | RLS policies for tenant-owned tables based on trusted session/context mechanism | Target/Unverified |
| Migration safety | Introduce RLS with rollout plan, superuser/admin considerations, and tests | Target/Unverified |
| App integration | Application sets required session/context safely; no SQL injection path | Target/Unverified |
| Exceptions | Any bypass (maintenance/admin jobs) is explicit, minimal, audited | Target/Unverified |

## Critical Risks
| Risk | Impact | Mitigation |
|---|---|---|
| Missing tenant filter in query/use case | Cross-tenant data exposure/modification | Mandatory tenant checks, code review, automated negative tests, RLS |
| RLS misconfiguration or bypass | Defense-in-depth failure | Migration review, policy tests, privileged-role controls |
| Background job without tenant context | Wrong-tenant writes or broad data access | Job contracts require tenant scope; validation and audit |
| Cache/key collisions across tenants | Data leakage or stale cross-tenant responses | Tenant-aware cache keys/namespaces |
| Webhook tenant mis-mapping | Wrong-tenant ingestion | Verified mapping config, signature verification, reject ambiguous events |

## Verification Requirements
- Negative API/integration tests: tenant A cannot access tenant B data.
- Use-case/data-access tests for tenant scoping.
- RLS policy tests (once implemented).
- Job/webhook tests proving tenant context is required and validated.
- Security review of admin/control-plane exceptions.

## Related Files
- `../07-security/05-authorization.md`
- `../07-security/01-threat-model.md`
- `08-lansweeper-integration.md`
- `../04-testing/07-security-testing.md`
