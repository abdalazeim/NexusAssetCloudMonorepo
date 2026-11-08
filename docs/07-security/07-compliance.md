# NexusAsset Cloud — Audit, Retention, and Compliance Controls

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (target design) |
| Evidence level | Target/Unverified; no compliance certification claimed |
| Primary evidence | `docs/07-security/*`, `docs/06-operations/*`, `docs/01-analysis/*`, `prompts/10-security-and-compliance.md` |

## Purpose
Define NexusAsset Cloud controls that support auditability, evidence retention, and compliance-oriented operations. This document does not claim certification (SOC 2, ISO 27001, GDPR compliance, etc.) unless formally assessed and evidenced.

## Control Areas (Target)
| Area | Control expectation | Evidence/status |
|---|---|---|
| Audit events | Significant auth, authorization, tenant, asset, sync, config, and admin actions generate audit records | Target/Unverified |
| Log retention | Logs/metrics/traces retained per approved policy; sensitive data minimized/redacted | Target/Unverified |
| Data retention | Tenant/business data retention and deletion/export workflows defined as required | Target/Unverified |
| Access reviews | Periodic review of privileged access and service credentials | Planned/Unverified |
| Change management | PR/ADR/release/approval records for significant changes | Partial process docs / Unverified enforcement |
| Incident evidence | Incident timeline, containment actions, RCA, corrective actions retained | Target/Unverified |
| Vendor/integration | Lansweeper and other integrations assessed for data flow, auth, and failure handling | Target/Unverified |

## Compliance Documentation Rules
1. Use precise language: "supports", "designed to support", or "evidenced by" — never claim certification without assessment evidence.
2. Link controls to owners, implementation artifacts, and test/operational evidence.
3. Record exceptions with compensating controls and review dates.

## Related Files
- `01-threat-model.md`
- `03-secrets-management.md`
- `../06-operations/08-incident-response.md`
- `../06-operations/09-backup-dr.md`
