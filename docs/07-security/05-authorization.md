# NexusAsset Cloud — Authorization and Access Control

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (target design) |
| Evidence level | Target/Unverified |
| Primary evidence | `docs/07-security/04-authentication.md`, `docs/02-design/09-multi-tenancy.md`, `docs/01-analysis/*`, `prompts/10-security-and-compliance.md` |

## Purpose
Define NexusAsset Cloud authorization: roles/permissions, tenant scope, policy enforcement points, administrative access, and verification requirements.

## Authorization Principles
1. **Deny by default** for protected resources.
2. **Server-side enforcement** — never rely on UI/mobile hiding alone.
3. **Tenant-first scoping** — authorization decisions must consider tenant membership/context for tenant-owned data.
4. **Least privilege** — roles/permissions grant minimum required capabilities.
5. **Auditable privileged actions** — admin/sensitive operations produce audit evidence.

## Target Policy Model
| Model element | Description | Status |
|---|---|---|
| User identity | Authenticated principal from `nxa_accounts` context | Target |
| Tenant membership/context | User's allowed tenant scope; resolved server-side | Target |
| Roles | Named role sets (e.g., admin, manager, auditor, operator) as approved | Target/Unverified |
| Permissions | Fine-grained actions/resources if implemented (RBAC/ABAC hybrid possible) | Target/Unverified |
| Resource ownership/scope | Tenant/resource attributes used in policy checks | Target |

## Enforcement Points (Target)
| Layer | Responsibility |
|---|---|
| Edge/BFF | Coarse protections, routing/session controls; not sole authorization authority |
| API interfaces | Authenticate request, extract validated principal, reject malformed/unauthorized requests |
| Application/use cases | Enforce permissions and tenant/resource rules before state changes |
| Data access | Tenant-aware queries and PostgreSQL RLS as defense in depth |
| Background jobs | Carry validated tenant/job context; never process cross-tenant data implicitly |

## Critical Scenarios
| Scenario | Expected result | Status |
|---|---|---|
| Tenant A user requests Tenant B asset | Deny; no data leakage; safe audit/log | Target/Unverified |
| User lacks permission for action | Deny (403/authorized failure per API policy) | Target/Unverified |
| Admin action on sensitive config | Allowed only by approved role; audited | Target/Unverified |
| Lansweeper sync job runs for tenant | Uses approved tenant/job context; writes scoped records only | Target/Unverified |
| Webhook triggers processing | Verification + tenant mapping rules enforced; no implicit privilege escalation | Target/Unverified |

## Verification Requirements
- RBAC/permission unit and integration tests.
- Tenant isolation negative tests across API/use-case/data layers.
- PostgreSQL RLS policy tests where applicable.
- Audit assertions for privileged/sensitive actions.

## Related Files
- `04-authentication.md`
- `../02-design/09-multi-tenancy.md`
- `../04-testing/07-security-testing.md`
