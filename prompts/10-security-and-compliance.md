# Phase 10 — Security and Compliance

## Objective
Apply NexusAsset Cloud security hardening and compliance controls across code, APIs, data, infrastructure, CI/CD, and operations.

## Preconditions
- Phases 01–09 provide implemented surfaces to secure.

## Required Context
- `SECURITY.md`, `.env.example`
- Backend auth/tenant/RLS code
- Traefik/Docker/Jenkins/Ansible configs
- `07-security/` docs
- Dependency manifests and CI scan configuration

## Deliverables
1. **Identity and access**
   - JWT/auth hardening, RBAC/permissions model, least privilege, admin separation, and secure session/token handling in BFF/mobile contexts.
   - Tenant isolation enforcement in application queries and database/RLS strategy; tests for cross-tenant access denial.
2. **Secrets and configuration**
   - Ensure all secrets use `NXA_*` names and are sourced from env/secret stores/vaults/CI credentials.
   - Remove/default safely any committed secret-like values; add/verify `.gitignore` and secret scanning.
3. **Application security**
   - Input validation, output encoding where relevant, secure file handling, CSRF/CORS policy, security headers, rate limiting, dependency hygiene, and audit logging for sensitive actions.
4. **Infrastructure and CI/CD security**
   - Harden Traefik TLS/headers/middleware, container user/capabilities/images, Jenkins credentials/plugins/agents, and Ansible privilege boundaries.
   - Define SAST/DAST/dependency/container/secret scan gates and triage expectations.
5. **Compliance/readiness artifacts**
   - Update threat model, control matrix, data handling notes, and incident/security runbook references.
   - Document evidence locations for audits (logs, scans, approvals, ADRs).

## Verification
┌─ Specification Block (converted from `powershell`; scope 2 — no fenced code blocks) ─┐
│ # Run available scanners/checks
│ # gitleaks detect --no-git or equivalent
│ # bandit/ruff security rules, safety/pip-audit, npm audit/pnpm audit, trivy fs/image
│ # backend tests for RBAC and tenant isolation
│ # OpenAPI security schemes/auth requirements review
└──────────────────────────────────────────────────────────────────────────────┘
Record findings, false positives, and remediations. Do not ignore critical/high issues without documented justification.

## Exit Criteria
- No committed secrets; secret handling is standardized on `NXA_*`.
- Auth/RBAC/tenant isolation controls are implemented and tested.
- CI/local security checks are defined and runnable.
- Security/compliance docs match the actual implementation.
