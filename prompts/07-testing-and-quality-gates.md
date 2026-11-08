# Phase 07 — Testing and Quality Gates

## Objective
Establish comprehensive automated testing and quality gates across backend, frontend, contracts, architecture, and security-sensitive paths.

## Preconditions
- Phases 01–06 implementation baselines exist.

## Required Context
- `tests/` (architecture, contract, e2e, load, security)
- `backend/` tests and tooling
- `frontend/` tests and tooling
- `mobile/android/` tests
- `api/contracts/openapi.yaml`
- `04-testing/` docs and CI expectations

## Deliverables
1. **Backend tests**
   - Unit tests for domain/application logic (especially `nxa_lansweeper` normalization/conflict/idempotency).
   - Integration tests for repositories/adapters/API with test DB/mocks.
   - API tests for auth, tenant scoping, pagination, and error contracts.
2. **Architecture/contract tests**
   - Dependency rule/import-linter or AST tests preventing domain/framework coupling.
   - OpenAPI validation and contract compatibility checks.
   - Codegen drift checks where generated artifacts are committed.
3. **Frontend/mobile tests**
   - Frontend unit/component tests, BFF route tests where feasible, and smoke E2E baseline.
   - Android unit tests for core data/domain logic; smoke/instrumentation scaffolding.
4. **Security/quality checks**
   - Lint/type/format gates: ruff/black/mypy (backend), eslint/tsc (frontend/workspaces), Kotlin lint where configured.
   - Dependency/secret/SAST baseline commands documented or wired (Trivy/Bandit/Safety/Gitleaks as applicable).
5. **Quality gate definition**
   - Update Makefile/scripts/CI-ready local commands so a single gate command set can be run.
   - Define DoD: required checks for PR, staging, and release.

## Verification
┌─ Specification Block (converted from `powershell`; scope 2 — no fenced code blocks) ─┐
│ # examples; use actual configured commands
│ make nxa-lint
│ make nxa-typecheck
│ make nxa-test-backend
│ make nxa-test-frontend
│ make nxa-openapi-validate
│ make nxa-arch-check
└──────────────────────────────────────────────────────────────────────────────┘
Capture pass/fail status for each gate. Fix failures caused by current phase changes.

## Exit Criteria
- Critical paths have automated tests.
- Architecture and contract gates exist.
- Local quality gate commands are deterministic and documented.
- Known gaps are explicitly recorded with risk and owner.
