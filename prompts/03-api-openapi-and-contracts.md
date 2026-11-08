# Phase 03 — API, OpenAPI 3.1, and Contracts

## Objective
Make `api/contracts/openapi.yaml` the complete, valid OpenAPI 3.1 source of truth and align backend/BFF/mobile-facing API contracts.

## Preconditions
- Phase 02 backend baseline exists.
- Contract file exists at `api/contracts/openapi.yaml`.

## Required Context
- `api/contracts/openapi.yaml`
- Backend DRF URLs/views/serializers/auth
- `09-api/` docs
- `frontend/` BFF/API client expectations (if present)
- `mobile/android/` API expectations (if present)

## Deliverables
1. **OpenAPI 3.1 contract expansion**
   - Define servers (`https://api.nexusasset.cloud`), tags, security schemes (JWT Bearer), common parameters, pagination, and error schemas.
   - Add core paths at minimum: health/readiness, auth/session/token bootstrap as applicable, tenants/current context, assets CRUD/list/filter, Lansweeper integration/sync status endpoints, audit/events read paths as appropriate, and mobile bootstrap endpoints.
   - Use consistent resource naming, HTTP status codes, problem/error responses, and idempotency headers where needed.
2. **Backend contract alignment**
   - Align DRF serializers/views/routes with OpenAPI operation IDs, request/response schemas, auth requirements, and tenant scoping.
   - Implement/align pagination, filtering, sorting, and error envelope conventions.
3. **Code generation/contracts workflow**
   - Add or align `nxa-codegen`/scripts to generate TypeScript types/client artifacts for `@nexusasset/api-client` or frontend BFF consumption.
   - Add OpenAPI validation command/script (`nxa-openapi-validate`).
4. **Compatibility rules**
   - Document versioning/deprecation policy (URL or header strategy) in `09-api/` and contract description.
   - Ensure breaking changes require explicit contract updates.

## Verification
┌─ Specification Block (converted from `powershell`; scope 2 — no fenced code blocks) ─┐
│ # OpenAPI validation (use available validator, e.g., spectral/openapi-cli/swagger-cli/redocly)
│ # nxa-openapi-validate
│ cd backend
│ python manage.py check
│ pytest -q tests  # include API/contract tests if added
│ # run codegen and ensure generated artifacts are up to date
└──────────────────────────────────────────────────────────────────────────────┘

## Exit Criteria
- OpenAPI 3.1 file is syntactically valid and internally consistent.
- Implemented/stubbed API routes match the contract's critical paths.
- Auth, tenancy, pagination, and error behavior are documented and reflected in schemas.
- Frontend/mobile consumers have a clear contract and codegen path.
