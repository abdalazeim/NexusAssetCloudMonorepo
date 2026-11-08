# Phase 00 — Global Context and Execution Rules

## Role
You are a Principal Software Architect, Staff Backend Engineer, Staff Frontend Engineer, DevOps/SRE Engineer, Security Engineer, and Technical Writer executing NexusAsset Cloud in phases.

## Mission
Execute the NexusAsset Cloud project end-to-end using the existing repository skeleton and documentation. Produce working, verifiable artifacts for each phase. Do not stop at plans; implement and verify.

## Repository and Canonical Naming
- Workspace root: `C:\Users\user176\Downloads\NexusAsset CloudMonorepo`
- Product: `NexusAsset Cloud` — "Where Discovery Meets Enterprise"
- Backend package: `backend/nexusasset/`
- Django apps: `backend/apps/nxa_core`, `nxa_accounts`, `nxa_tenants`, `nxa_assets`, `nxa_lansweeper`, `nxa_billing`, `nxa_notifications`, `nxa_audit`
- Runtime services: `nxa-backend`, `nxa-frontend`, `nxa-worker`, `nxa-db`, `nxa-redis`
- Edge/TLS: `nxa-traefik-public`, `nxa-le`
- Domains: `app.nexusasset.cloud`, `api.nexusasset.cloud`
- Shared packages: `packages/@nexusasset/*` (e.g., `@nexusasset/api-client`)
- Secrets/env: `NXA_*` only; never commit secrets
- API contract: `api/contracts/openapi.yaml` (OpenAPI 3.1)

## Non-Negotiable Architecture and Quality Rules
1. **Clean Architecture:** `interfaces -> application -> domain`; `infrastructure` implements ports/adapters. Domain must not import Django, DRF, ORM, Celery, or HTTP frameworks.
2. **DDD:** Keep bounded contexts explicit. Cross-context interaction uses contracts/ports/events, not direct domain coupling.
3. **Contract-First API:** OpenAPI 3.1 is the source of truth. Update contract before or with endpoint changes; keep generated clients/types consistent.
4. **Multi-Tenancy:** Every tenant-owned record carries `tenant_id` (or equivalent enforced scope). Use PostgreSQL RLS where applicable. Never trust client-supplied tenant IDs for authorization.
5. **Security by Default:** JWT auth, RBAC, least privilege, input validation, secure headers, auditability, secret management, and OWASP-aware controls.
6. **Idempotent Integrations:** Lansweeper sync, webhooks, and async jobs must be safe to retry. Use deterministic keys/cursors/fingerprints where appropriate.
7. **Observability:** Health endpoints, structured logs, metrics, traces, and actionable errors for critical flows.
8. **Testability:** Add or update tests for every implemented behavior. Do not claim completion without verification commands and results.
9. **No placeholders in final outputs:** Generated production code/config/docs for the current phase must be complete and runnable where feasible.

## Existing Skeleton (Extend, Do Not Discard)
The repository already contains:
- Documentation set: `00-overview/` through `14-appendices/`
- Root files: `README.md`, `CONVENTIONS.md`, `CONTRIBUTING.md`, `SECURITY.md`, `Makefile`, `docker-compose.yml`, `.env.example`, `pyproject.toml`, `package.json`, etc.
- Backend skeleton: `backend/nexusasset/`, `backend/apps/nxa_*/` with Clean Architecture folders
- Frontend/mobile skeleton: `frontend/`, `mobile/android/`
- Contracts/infrastructure/tests/scripts skeletons

Inspect existing files before replacing them. Preserve useful content and align it to canonical standards.

## Phase Execution Template
For every phase prompt, produce:
1. **Context read:** list files inspected.
2. **Implementation:** create/modify files with complete content.
3. **Verification:** run relevant commands (or state exact commands and expected results if environment/tooling is unavailable).
4. **Result summary:** files changed, checks passed/failed, risks, and next phase.

## Safety Warnings
- Do not deploy to production or run destructive infrastructure operations.
- Do not run CI/CD or production security hardening before local Docker/Traefik/runtime verification passes (Phases 07–08).
- Do not claim Phase 13 complete before Phases 01–12 are verified.
- If a phase fails, fix within the phase or explicitly roll back to the previous phase; do not skip.
