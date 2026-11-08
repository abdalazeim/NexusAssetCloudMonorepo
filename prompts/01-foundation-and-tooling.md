# Phase 01 — Foundation and Tooling

## Objective
Turn the existing NexusAsset Cloud monorepo skeleton into a complete, consistent foundation for backend, frontend, mobile, contracts, infrastructure, tests, and automation.

## Preconditions
- Phase 00 rules are understood.
- Workspace is the repository root.
- Existing skeleton files may be present; inspect before editing.

## Required Context
Read/inspect as needed:
- `README.md`, `CONVENTIONS.md`, `CONTRIBUTING.md`, `SECURITY.md`
- `.env.example`, `Makefile`, `docker-compose.yml`
- `pyproject.toml`, `package.json`, `pnpm-workspace.yaml`, `turbo.json`
- `backend/pyproject.toml`, `backend/manage.py`, `backend/Dockerfile`
- `frontend/package.json`, `frontend/README.md`
- `api/contracts/openapi.yaml`
- `03-implementation/01-project-structure.md`

## Deliverables
1. **Root governance/config alignment**
   - Ensure root configs use canonical `NXA_*`, `nxa-*`, and NexusAsset Cloud naming.
   - Align `.env.example` with local/dev/test/staging/prod-safe variables (no real secrets).
   - Align `Makefile` targets to canonical `nxa-*` operations (`nxa-dev`, `nxa-test-backend`, `nxa-test-frontend`, `nxa-lint`, `nxa-typecheck`, `nxa-codegen`, `nxa-openapi-validate`, `nxa-arch-check`, etc.).
2. **Backend tooling baseline**
   - Complete `backend/pyproject.toml` (or equivalent) for Django 5, DRF, pytest, ruff, black, mypy, import-linter, celery, redis, psycopg, OpenAPI tooling as appropriate.
   - Ensure settings layout supports `base/dev/test/staging/prod` or clearly document the selected equivalent.
   - Ensure `manage.py`, ASGI/WSGI/Celery entrypoints, and dependency injection/composition root skeletons are coherent.
3. **Frontend/mobile/packages baseline**
   - Ensure `frontend/` has a runnable Next.js 15/React 19 TypeScript baseline (package scripts, tsconfig, lint/typecheck placeholders as needed).
   - Ensure `mobile/android/` has a documented Kotlin/Gradle baseline structure.
   - Ensure `packages/@nexusasset/*` workspace manifests are consistent with pnpm/turbo.
4. **Contract and automation baseline**
   - Keep `api/contracts/openapi.yaml` as OpenAPI 3.1 source of truth with a minimal valid spec.
   - Add/align scripts under `scripts/` (or documented equivalents) for setup, lint, test, codegen, and local checks.
5. **Documentation alignment**
   - Update affected docs/READMEs when structure or commands change.

## Verification
Run or provide exact commands and outcomes:
┌─ Specification Block (converted from `powershell`; scope 2 — no fenced code blocks) ─┐
│ # Structure and config sanity
│ Get-ChildItem backend,frontend,mobile,packages,api,infrastructure,tests,scripts
│ # Backend tooling (if Python env available)
│ cd backend; python -m compileall .; python manage.py check --settings=nexusasset.settings.dev  # adjust if settings path differs
│ # Frontend/package sanity (if Node/pnpm available)
│ cd ..; pnpm -v; pnpm install --lockfile-only  # or documented equivalent
│ # OpenAPI sanity
│ # validate api/contracts/openapi.yaml with available tool
└──────────────────────────────────────────────────────────────────────────────┘
If a tool is unavailable, report the limitation and provide a deterministic fallback check.

## Exit Criteria
- Canonical naming is consistent across configs and docs.
- Local developer commands are documented and internally consistent.
- Backend/frontend/contracts foundations are ready for implementation phases.
- No secrets are committed.
