# NexusAsset Cloud — Execution Prompt Pack

This directory contains the canonical, professional English prompt pack for executing the entire NexusAsset Cloud project in phases. Each prompt is designed to be run in a fresh AI session, produce repository artifacts, and pass explicit verification before moving to the next phase.

## Execution Rules
1. Start each phase in a new AI session.
2. Read the required context files before editing.
3. Do not skip verification. If verification fails, fix issues in the same phase and rerun verification.
4. Save outputs to the repository before continuing.
5. Use `continue` if output is truncated.
6. Do not invent undocumented names: use canonical NexusAsset Cloud naming only.
7. Do not run CI/CD hardening or production deployment prompts before local Docker/Traefik verification passes.
8. Treat OpenAPI as the API source of truth; do not generate ad hoc endpoints without updating the contract.

## Canonical Standards
- Product: `NexusAsset Cloud` — "Where Discovery Meets Enterprise"
- Repository: `nexusasset-cloud/` (current workspace root)
- Backend package: `backend/nexusasset/`
- Django apps: `nxa_core`, `nxa_accounts`, `nxa_tenants`, `nxa_assets`, `nxa_lansweeper`, `nxa_billing`, `nxa_notifications`, `nxa_audit`
- Runtime services: `nxa-backend`, `nxa-frontend`, `nxa-worker`, `nxa-db`, `nxa-redis`
- Edge/TLS: `nxa-traefik-public`, `nxa-le`
- Domains: `app.nexusasset.cloud`, `api.nexusasset.cloud`
- Packages: `@nexusasset/*`
- Secrets: `NXA_*` only (never committed)
- Contract: `api/contracts/openapi.yaml` (OpenAPI 3.1)
- Architecture: Clean Architecture + DDD; dependency rule inward only; domain has no Django/ORM imports
- Data isolation: `tenant_id` + PostgreSQL Row-Level Security (RLS) where applicable
- Integration: Lansweeper sync must be idempotent, observable, rate-limit aware, and Field Ownership Matrix compliant

## Prompt Index

| Phase | File | Objective | Verification Gate |
|---:|---|---|---|
| 00 | `00-global-context-and-rules.md` | Global execution context, non-negotiables, safety | Rules acknowledged in every phase |
| 01 | `01-foundation-and-tooling.md` | Complete monorepo foundation, tooling, env, scripts, local commands | Repo structure/tooling/config files are consistent |
| 02 | `02-backend-clean-architecture.md` | Implement Django/DRF backend skeleton into working Clean Architecture baseline | Backend boots/checks pass; architecture boundaries enforced |
| 03 | `03-api-openapi-and-contracts.md` | Expand OpenAPI 3.1 and align DRF/BFF/mobile contracts | OpenAPI valid; routes/serializers/codegen aligned |
| 04 | `04-lansweeper-integration.md` | Implement Lansweeper bounded context (sync, webhooks, delta, bulk, conflicts) | Idempotent sync paths and tests pass |
| 05 | `05-frontend-bff.md` | Implement Next.js 15 + React 19 + BFF web application baseline | Frontend builds; BFF/API integration works |
| 06 | `06-mobile-android.md` | Implement Android Kotlin client baseline against API contracts | Mobile build/tests pass; API bootstrap/sync flows wired |
| 07 | `07-testing-and-quality-gates.md` | Implement unit/integration/contract/E2E/architecture/security test gates | All required quality gates pass locally |
| 08 | `08-devops-docker-traefik.md` | Complete local/prod-ready Docker Compose, Traefik, TLS, networking, health checks | `nxa-*` services start; routing/TLS/health checks pass |
| 09 | `09-cicd-jenkins-ansible.md` | Implement Jenkins pipelines and Ansible deployment automation | Pipeline stages/gates defined; staging deploy path verified |
| 10 | `10-security-and-compliance.md` | Apply hardening, secrets, RBAC, RLS validation, scanning, compliance controls | Security scans and tenant/auth checks pass |
| 11 | `11-observability-and-operations.md` | Implement metrics, logs, traces, dashboards, alerts, runbooks, backup/DR hooks | Observability stack and operational checks pass |
| 12 | `12-documentation-adr-runbooks.md` | Finalize docs, ADRs, runbooks, diagrams, release/readiness artifacts | Docs are accurate, linked, and implementation-aligned |
| 13 | `13-final-verification-and-release.md` | Run full checklist, produce Go/No-Go report, and prepare release handoff | All critical gates pass; release report generated |

## Current Workspace Notes
- The workspace already contains a documentation tree (`00-overview`–`14-appendices`), root governance files, and an initial executable monorepo skeleton (`backend/`, `frontend/`, `mobile/`, `packages/`, `api/`, `infrastructure/`, `tests/`, `scripts/`).
- Prompts must extend and harden the existing skeleton; do not discard useful existing files without documenting the reason.
