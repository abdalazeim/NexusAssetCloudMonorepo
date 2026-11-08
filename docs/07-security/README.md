# NexusAsset Cloud — Security Architecture and Controls

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Baseline documentation expanded (Phase B/C in progress) |
| Evidence level | Implemented (Descriptor Evidence) + Target; no runtime security proof claimed |
| Primary evidence | `project-meta/SECURITY.md`, `project-meta/.env.example.description.md`, `prompts/10-security-and-compliance.md`, `docs/07-security/*`, ADRs/analysis/design docs |
| Constraint | This repository is in a specification/execution foundation state; do not present controls as verified runtime behavior without tests/scans/config evidence. |

## Purpose
This module defines NexusAsset Cloud security architecture, control objectives, trust boundaries, and evidence requirements. It covers identity, authorization, tenant isolation, API/BFF security, secrets, integration security (including Lansweeper), infrastructure security, auditability, vulnerability management, and incident response.

## Security Objectives
1. Protect confidentiality, integrity, and availability of tenant asset and business data.
2. Prevent cross-tenant data access (critical for CMDB/ITAM SaaS).
3. Enforce least privilege for users, services, pipelines, and operators.
4. Ensure secrets are managed through `NXA_*` references and approved secret stores; never committed.
5. Make significant security and business actions auditable (`nxa_audit`).
6. Detect and remediate vulnerabilities through quality/security gates before promotion.

## Trust Boundaries (Target)
| Boundary | Description | Primary controls (Target/Descriptor) |
|---|---|---|
| Public internet to edge | External users/clients reach `app.nexusasset.cloud` / `api.nexusasset.cloud` via `nxa-traefik-public` | TLS, secure headers, rate limits, routing restrictions |
| Web/BFF to backend API | BFF aggregates and forwards authorized requests | Auth/session/JWT handling, server-side token handling, contract alignment |
| Backend API to data stores | `nxa-backend`/`nxa-worker` access PostgreSQL/Redis | `NXA_*` credentials, network segmentation, least privilege, tenant-aware queries, RLS target |
| Backend/worker to Lansweeper | External discovery integration | Approved auth mode, retries/backoff, rate-limit handling, webhook verification, idempotency |
| CI/CD to environments | Jenkins/Ansible deployment path | Credentials/secrets store, approvals, gated promotion, audit trail |
| Operator/admin access | Privileged operational actions | MFA (target), RBAC, break-glass process (target), logging/audit |

## Identity and Access Control (Target)
| Area | Required documentation/implementation detail | Status |
|---|---|---|
| Authentication | Login lifecycle, credential validation, user status checks, token issuance/claims/expiry, refresh lifecycle, revocation/rotation if used | Target / Unverified |
| Authorization | RBAC roles/permissions, policy evaluation points, admin separation, least privilege | Target / Unverified |
| Tenant context | Tenant resolution source of truth, propagation, and enforcement; never trust client-supplied tenant for authorization | Target / Unverified |
| BFF/session | Cookie/session strategy, CSRF/CORS, token exposure minimization | Target / Unverified |
| Mobile auth | Token storage, refresh, device/session controls, logout/revocation behavior | Target / Unverified |

## Tenant Isolation and Data Protection (Critical)
| Control | Requirement | Evidence/status |
|---|---|---|
| Data model scoping | Tenant-owned records carry `tenant_id` or equivalent enforced scope | Target; schema/migrations unverified |
| Application enforcement | Every query/use case/path enforces tenant authorization | Target; tests unverified |
| PostgreSQL RLS | Policies enforce tenant isolation at database layer where applicable | Target; no verified RLS policies in current state |
| Cross-tenant tests | Automated negative tests prove access denial | Target; unverified |
| Audit | Tenant-sensitive operations generate audit events | Target; unverified |

## Secrets and Configuration Security
| Control | Requirement | Evidence/status |
|---|---|---|
| Naming | Use `NXA_*` for NexusAsset Cloud secrets/config keys | Implemented (Descriptor Evidence): `.env.example.description.md`, prompts/docs |
| Storage | Use local uncommitted env for dev and approved secret stores/vault/CI credentials for shared/prod environments | Target |
| Repository hygiene | No real credentials/tokens/private keys committed | Target control; enforce with secret scanning |
| Rotation | Define rotation procedure for JWT/signing keys, DB credentials, Lansweeper tokens, CI credentials | Target; runbook needed |
| Exposure | Do not print secrets in logs, CI output, or error messages | Target |

## Application and API Security (Target)
- Input validation at interfaces and use-case boundaries.
- Consistent error responses without sensitive internals.
- Pagination/filter/sort limits to reduce abuse/resource exhaustion.
- Rate limiting at edge and/or application as appropriate.
- CSRF/CORS policy aligned to BFF/browser flows.
- Secure file/upload handling if implemented.
- Webhook verification for Lansweeper/integration callbacks.
- Idempotency for retried integration and async operations.

## Infrastructure and CI/CD Security (Target)
| Area | Control objective | Status |
|---|---|---|
| Containers | Minimal images, non-root where feasible, no baked secrets, vulnerability scanning | Target; Docker descriptors exist |
| Traefik | TLS, HSTS/security headers, controlled dashboard access, routing constraints | Target |
| Jenkins | Credential store, least privilege, plugin governance, protected branches/promotion approvals | Target; Jenkins descriptor exists |
| Ansible | Idempotent playbooks, vault/secret references, limited privilege escalation | Target |
| Dependencies | Dependency and container scanning with triage policy | Target |

## Vulnerability and Compliance Management
| Activity | Expected output | Status |
|---|---|---|
| Secret scanning | Findings report, remediation, prevention | Target / Unverified |
| SAST | Findings by severity, triage decisions | Target / Unverified |
| Dependency/container scan | CVE findings, patch/mitigation plan | Target / Unverified |
| DAST (staging) | Runtime findings for exposed surfaces | Target / Unverified |
| Threat model | Assets, trust boundaries, threats, mitigations, residual risk | Missing detailed artifact; create in Phase B/C |
| Controls matrix | Control → implementation → evidence → owner → status | Missing; create in Phase B/C |

## Incident Response and Audit (Target)
- Security-relevant events must produce audit records where appropriate.
- Logs must support investigation without leaking secrets or unnecessary sensitive data.
- Define severity, triage, containment, eradication, recovery, and post-incident review process.
- Lansweeper token compromise, tenant-access anomaly, CI credential leak, and webhook abuse require specific runbooks.

## Contents
| File | Description | Audit note |
|---|---|---|
| `01-threat-model.md` | Threat model and trust boundaries | Expand with assets, STRIDE-style analysis, mitigations, residual risk. |
| `02-owasp-checklist.md` | OWASP-oriented checklist | Map each item to NexusAsset Cloud controls and evidence status. |
| `03-secrets-management.md` | Secrets management and rotation | Expand with lifecycle, storage options, rotation, incident handling. |
| `04-authentication.md` | Authentication flows and controls | Expand with end-to-end lifecycle and status labels. |
| `05-authorization.md` | Authorization and RBAC/ABAC | Expand with policy model, tenant enforcement, tests. |
| `06-mobile-security.md` | Mobile security controls | Expand with token/session/storage/transport controls. |
| `07-compliance.md` | Audit, retention, compliance controls | Expand with evidence artifacts and operational responsibilities. |

## Required Phase B/C Additions
- `docs/07-security/SECURITY_CONTROLS_MATRIX.md` (control mapping).
- `docs/07-security/THREAT_MODEL.md` or deep expansion of `01-threat-model.md`.
- Tenant isolation/RLS test specification cross-linked to `docs/04-testing/` and `docs/02-design/09-multi-tenancy.md`.

## Related Files
- `../README.md`
- `../02-design/09-multi-tenancy.md`
- `../02-design/08-lansweeper-integration.md`
- `../04-testing/07-security-testing.md`
- `../06-operations/08-incident-response.md`
- `../../project-meta/SECURITY.md`
- `../../project-meta/.env.example.description.md`
