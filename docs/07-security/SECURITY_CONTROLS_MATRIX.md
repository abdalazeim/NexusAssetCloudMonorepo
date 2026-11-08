# NexusAsset Cloud — Security Controls Matrix

## Status
Target/planning matrix. Rows are seeded controls; implementation and evidence columns must be completed during delivery. Do not mark controls Verified without evidence.

| Control ID | Control | Domain | Expected implementation | Evidence required | Owner | Status |
|---|---|---|---|---|---|---|
| SEC-001 | `NXA_*` secrets only; no committed secrets | Secrets | Env/secret store integration, templates, scanning | `.gitignore`/scan config + scan results + config refs | Platform/Security | Descriptor/Target |
| SEC-002 | JWT/BFF authentication lifecycle | Identity | Auth endpoints, token issuance/validation, expiry/refresh policy | API tests, security review, config | `nxa_accounts`/Web | Target/Unverified |
| SEC-003 | RBAC/least privilege | Authorization | Role/permission model, enforcement points, admin separation | Policy docs + negative tests | `nxa_accounts`/Security | Target/Unverified |
| SEC-004 | Tenant isolation in application layer | Multi-tenancy | Tenant context resolution, scoped queries/use cases | Integration/negative tests | `nxa_tenants`/all contexts | Target/Unverified |
| SEC-005 | PostgreSQL RLS for tenant-owned tables | Data | RLS policies/migrations, DB role/session settings | Migration + SQL/policy tests | DB/Platform | Target/Unverified |
| SEC-006 | Lansweeper webhook verification | Integration | Signature/secret/timestamp validation, replay protection | Unit/integration tests | `nxa_lansweeper` | Target/Unverified |
| SEC-007 | Lansweeper sync idempotency/Field Ownership Matrix | Integration/data integrity | Dedupe keys, upsert rules, ownership/conflict policy | Unit/integration tests, audit records | `nxa_lansweeper`/`nxa_assets` | Target/Unverified |
| SEC-008 | Edge TLS/headers/rate limits | Network/edge | Traefik TLS, HSTS/security headers, rate limits | Config refs + runtime checks | DevOps/Security | Target/Unverified |
| SEC-009 | Audit logging for sensitive actions | Audit/compliance | `nxa_audit` events, structured logs, redaction | Event tests + log review | `nxa_audit`/Platform | Target/Unverified |
| SEC-010 | CI/CD protected delivery | Supply chain | Credential store, approvals, quality/security gates | Pipeline config + run evidence | DevOps | Descriptor/Target |
| SEC-011 | Dependency/container/SAST/DAST scanning | Vulnerability mgmt | Scanner integration and triage workflow | Reports + remediation tracking | Security/DevOps | Target/Unverified |
| SEC-012 | Backup/restore and incident response | Operations/resilience | Backup jobs, restore drills, IR/runbooks | Drill reports + runbooks | SRE/Platform | Target/Unverified |
