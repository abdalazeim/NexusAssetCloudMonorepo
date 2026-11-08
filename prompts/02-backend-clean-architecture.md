# Phase 02 — Backend Clean Architecture Baseline

## Objective
Implement a working Django 5 + DRF backend baseline in `backend/` that enforces NexusAsset Cloud Clean Architecture and bounded contexts.

## Preconditions
- Phase 01 complete.
- Canonical apps exist or are created under `backend/apps/`: `nxa_core`, `nxa_accounts`, `nxa_tenants`, `nxa_assets`, `nxa_lansweeper`, `nxa_billing`, `nxa_notifications`, `nxa_audit`.

## Required Context
- `backend/nexusasset/settings/*`
- `backend/nexusasset/urls.py`, `asgi.py`, `wsgi.py`, `celery.py`
- `backend/apps/nxa_*/` structure
- `CONVENTIONS.md`, `03-implementation/01-project-structure.md`
- `02-design/04-clean-architecture.md` (if present)

## Deliverables
1. **Project configuration**
   - Configure Django settings modules (base/dev/test/staging/prod or documented equivalent) using `NXA_*` environment variables.
   - Configure PostgreSQL and Redis settings placeholders, JWT/auth baseline, CORS/CSRF/security defaults, logging, and Celery app wiring.
2. **Clean Architecture enforcement**
   - For each `nxa_*` app, ensure `domain/`, `application/`, `infrastructure/`, `interfaces/`, `migrations/`, and `tests/` are usable.
   - Add architecture boundary checks (e.g., `import-linter` config and/or AST tests under `tests/architecture/`).
   - Domain layer must contain pure Python business constructs only.
3. **Core bounded-context baseline**
   - `nxa_core`: base entities/value objects/errors/result types, tenant-aware abstractions where appropriate.
   - `nxa_accounts`: user/auth domain/application ports and DRF/JWT interface skeleton.
   - `nxa_tenants`: tenant model/application flow skeleton and tenant context resolution.
   - `nxa_assets`: asset/CI domain skeleton and repository ports.
   - `nxa_audit`: audit event model/use-case skeleton.
   - Other apps: coherent placeholders with ports/use-case/interface entrypoints.
4. **Persistence and tenancy baseline**
   - Implement tenant-aware model/repository patterns.
   - Add migration strategy notes/code hooks for PostgreSQL RLS enablement (safe, environment-aware).
5. **API/interface baseline**
   - Add DRF routing for health/readiness and minimal authenticated endpoints as needed.
   - Ensure error handling returns consistent API error shapes (to be fully aligned in Phase 03).

## Verification
┌─ Specification Block (converted from `powershell`; scope 2 — no fenced code blocks) ─┐
│ cd backend
│ python -m compileall .
│ python manage.py check
│ python manage.py makemigrations --check --dry-run  # if migrations are expected
│ pytest -q  # or targeted tests
│ # run import-linter / architecture tests if configured
└──────────────────────────────────────────────────────────────────────────────┘

## Exit Criteria
- Backend starts/configures without import or settings errors in the supported local mode.
- Architecture checks exist and cover domain dependency violations.
- Bounded contexts have implementable Clean Architecture skeletons, not empty placeholders.
- Tenant/audit/security foundations are present for later phases.
