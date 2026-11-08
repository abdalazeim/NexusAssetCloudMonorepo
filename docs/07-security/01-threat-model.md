# NexusAsset Cloud — Threat Model and Trust Boundaries

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Baseline threat model (documentation) |
| Evidence level | Target/Unverified; no penetration test or runtime threat validation claimed |
| Primary evidence | `docs/07-security/*`, `docs/02-design/08-lansweeper-integration.md`, `docs/02-design/09-multi-tenancy.md`, `project-meta/SECURITY.md`, `prompts/10-security-and-compliance.md` |

## Purpose
Identify NexusAsset Cloud assets, trust boundaries, threat scenarios, required mitigations, and residual risks. This document is a design-level threat model and must be updated with implementation evidence.

## Assets (What We Protect)
| Asset | Sensitivity | Impact if compromised |
|---|---|---|
| Tenant asset/CMDB data | High | Cross-tenant exposure, integrity loss, compliance risk |
| Identity credentials/tokens | Critical | Account takeover, unauthorized access |
| `NXA_*` secrets (DB, JWT, Lansweeper tokens, CI credentials) | Critical | Full system/data compromise |
| Audit records | High | Loss of accountability/evidence |
| Integration credentials/webhook secrets | Critical | Unauthorized sync, data poisoning, replay abuse |
| Billing/commercial records | High | Financial/compliance impact |
| Operational telemetry | Medium/High | Information disclosure, blind spots if tampered |

## Trust Boundaries (Target)
| ID | Boundary | Entry points | Required controls |
|---|---|---|---|
| TB-01 | Public internet → edge | `app.nexusasset.cloud`, `api.nexusasset.cloud` via `nxa-traefik-public` | TLS, headers, rate limits, WAF/edge controls as approved |
| TB-02 | Web client → BFF | Browser/app requests | Auth/session, CSRF/CORS, input validation, no secret exposure |
| TB-03 | BFF/mobile → backend API | Internal/API calls | JWT/authZ, tenant scope, contract validation |
| TB-04 | Backend/worker → PostgreSQL/Redis | Data/queue access | Network segmentation, least privilege, secret management, RLS target |
| TB-05 | Backend/worker → Lansweeper | GraphQL/webhook/sync flows | Credential protection, verification, idempotency, retries/backoff, allowlist as appropriate |
| TB-06 | CI/CD → registries/environments | Jenkins/Ansible/deploy paths | Protected credentials, approvals, audit, least privilege |
| TB-07 | Admin/operator → infrastructure | Privileged operations | MFA/RBAC target, break-glass, logging, change control |

## Threat Analysis (Selected)
| Threat ID | Scenario | Boundary | Impact | Mitigation (Target) | Status |
|---|---|---|---|---|---|
| TM-001 | Cross-tenant data access via manipulated ID/filter | TB-03/TB-04 | Critical | Server-side tenant resolution, RBAC, `tenant_id` filters, PostgreSQL RLS, negative tests | Unverified |
| TM-002 | Token theft/replay/session hijack | TB-01/TB-02/TB-03 | High/Critical | Secure token handling, expiry/refresh controls, TLS, CSRF protections, revocation strategy | Unverified |
| TM-003 | Lansweeper token/credential leak | TB-05/TB-06 | Critical | `NXA_*` secret store, no commits/logging, rotation, scoped credentials | Unverified |
| TM-004 | Webhook spoofing/replay | TB-05 | High | Signature/secret verification, timestamp/replay window, idempotency keys | Unverified |
| TM-005 | Sync data poisoning/unsafe overwrite | TB-05 | High | Payload validation, Field Ownership Matrix, conflict handling, audit | Unverified |
| TM-006 | Injection (SQL/command/template) | TB-02/TB-03 | High | Parameterized queries/ORM safety, input validation, output encoding, SAST | Unverified |
| TM-007 | Secrets committed or exposed in logs/CI | TB-06/repo | Critical | Secret scanning, log redaction, CI masking, `.env` hygiene | Unverified |
| TM-008 | Privileged pipeline compromise | TB-06 | Critical | Protected branches, approvals, credential scoping, signed artifacts as adopted | Unverified |
| TM-009 | Insufficient audit evidence | TB-03/TB-07 | Medium/High | `nxa_audit` events for sensitive actions, immutable/log retention strategy | Unverified |
| TM-010 | Denial of service/resource exhaustion | TB-01/TB-03 | Medium/High | Rate limits, pagination limits, queue backpressure, timeouts, monitoring | Unverified |

## Residual Risks (Current)
- Runtime controls and tests are not verified in the current specification/descriptor state.
- PostgreSQL RLS policies, RBAC enforcement, and scan reports require implementation evidence.
- External Lansweeper API behavior and webhook capabilities must be validated against approved integration configuration.

## Required Evidence to Close Threats
- Security controls matrix with control owner and test/scan evidence.
- Tenant isolation and authorization negative tests.
- Secret scanning and dependency/SAST/DAST reports (as applicable).
- Lansweeper webhook verification/idempotency tests.
- Incident runbooks and audit verification.

## Related Files
- `README.md`
- `02-owasp-checklist.md`
- `03-secrets-management.md`
- `04-authentication.md`
- `05-authorization.md`
- `../02-design/08-lansweeper-integration.md`
- `../02-design/09-multi-tenancy.md`
