# NexusAsset Cloud — OWASP-Oriented Security Checklist

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Baseline checklist (documentation) |
| Evidence level | Target/Unverified unless mapped to verified implementation/tests/scans |
| Primary evidence | `docs/07-security/*`, `prompts/10-security-and-compliance.md`, `project-meta/SECURITY.md` |

## Purpose
Provide an OWASP-oriented checklist for NexusAsset Cloud design, implementation, and review. Items are controls to implement and verify, not claims of current compliance.

## Checklist
| ID | Area | NexusAsset Cloud control expectation | Evidence/status |
|---|---|---|---|
| OWASP-01 | Broken Access Control | Server-side RBAC, tenant scoping, PostgreSQL RLS target, deny-by-default for sensitive resources, negative tests | Target/Unverified |
| OWASP-02 | Cryptographic Failures | TLS in transit, approved hashing/encryption, secure secret storage, no sensitive data in logs | Target/Unverified |
| OWASP-03 | Injection | Parameterized queries/ORM safety, input validation, command/template safety, SAST | Target/Unverified |
| OWASP-04 | Insecure Design | Threat model, trust boundaries, abuse cases, tenant/Lansweeper failure scenarios | Partial (docs) / Unverified runtime |
| OWASP-05 | Security Misconfiguration | Hardened containers/edge/CI, secure defaults, no debug/prod secret leakage, config review | Target/Unverified |
| OWASP-06 | Vulnerable Components | Dependency/container scanning, patch policy, SBOM as adopted | Target/Unverified |
| OWASP-07 | Identification & Authentication Failures | Secure login/token lifecycle, MFA/admin controls target, brute-force/rate limits | Target/Unverified |
| OWASP-08 | Software & Data Integrity Failures | Protected CI/CD, dependency integrity, webhook verification, idempotent sync | Target/Unverified |
| OWASP-09 | Logging & Monitoring Failures | Structured logs, audit events, alerting, no secret logging, incident runbooks | Target/Unverified |
| OWASP-10 | SSRF / Unsafe Outbound Calls | Restrict/validate outbound integration targets, timeouts, allowlists/egress controls as appropriate | Target/Unverified |

## Review Instructions
1. For each item, link implementation evidence (code/config/policy), test evidence, and scan evidence.
2. Mark exceptions with risk owner, compensating control, and expiry/review date.
3. Do not mark an item complete without evidence.

## Related Files
- `01-threat-model.md`
- `04-authentication.md`
- `05-authorization.md`
- `../04-testing/07-security-testing.md`
