# Phase 04 — Lansweeper Integration

## Objective
Implement the `nxa_lansweeper` bounded context as a production-grade, idempotent integration that ingests Lansweeper discovery data into NexusAsset Cloud CMDB models.

## Preconditions
- Phases 01–03 complete: backend baseline, tenant handling, assets/domain ports, and API/contract patterns exist.

## Required Context
- `backend/apps/nxa_lansweeper/`
- `backend/apps/nxa_assets/`, `nxa_tenants/`, `nxa_audit/`, `nxa_core/`
- `api/contracts/openapi.yaml`
- `03-documentation-formatting-and-context-rules.md` equivalent guidance in docs (if needed)
- Any Lansweeper integration docs under `02-design/` or `01-analysis/`

## Deliverables
1. **Domain layer (pure Python)**
   - Entities/value objects: discovered asset, asset identity, relationship, sync job/run, sync cursor, source reference, conflict record.
   - Domain services/policies: normalization, identity matching, conflict resolution, Field Ownership Matrix evaluation.
   - Domain events/errors for discovery, updates, conflicts, rate limits, source failures.
2. **Application layer**
   - Use cases: connect/configure source, run full sync, run delta sync, ingest bulk export, handle webhook, resolve conflict, record sync status.
   - Ports: Lansweeper gateway, asset/CMDB repository, sync state repository, audit/event publisher, clock/idempotency store.
   - DTOs/commands/queries for API and tasks.
3. **Infrastructure layer**
   - Lansweeper adapter: GraphQL client, auth strategy abstraction (PAT/OAuth as configured), query files/modules, retry/backoff, rate-limit awareness, pagination/cursor handling.
   - Webhook verification, bulk export ingestion adapter, persistence mappers/repositories, Celery tasks and Beat schedules.
   - Safe configuration via `NXA_LANSWEEPER_*` env vars.
4. **Interfaces**
   - DRF endpoints for integration configuration/status (authorized), webhook receiver endpoint, and admin/ops sync triggers.
   - Management command for manual sync (e.g., `manage.py nxa_lansweeper_sync`).
5. **Field Ownership Matrix and idempotency**
   - Implement explicit ownership rules: discovery may update discovery-owned fields; business/manual fields require policy-based merge or conflict workflow.
   - All sync paths must be idempotent (safe retries, duplicate webhook/event suppression, deterministic external IDs/cursors).
6. **Observability and audit**
   - Emit structured logs/metrics around sync duration, records processed, conflicts, rate-limit events, and failures.
   - Write audit records for significant sync/configuration actions.

## Verification
┌─ Specification Block (converted from `powershell`; scope 2 — no fenced code blocks) ─┐
│ cd backend
│ python -m compileall apps/nxa_lansweeper
│ pytest apps/nxa_lansweeper -q
│ pytest tests -k lansweeper -q  # if cross-cutting tests exist
│ # verify webhook endpoint routing and OpenAPI alignment
└──────────────────────────────────────────────────────────────────────────────┘
Use mocks/fakes for external Lansweeper API calls. Do not require live Lansweeper credentials for tests.

## Exit Criteria
- `nxa_lansweeper` has working Clean Architecture layers.
- Full/delta/bulk/webhook ingestion paths exist with idempotency controls.
- Field Ownership Matrix prevents unsafe overwrite of business-owned data.
- Tests cover normalization, conflict resolution, and gateway/task behavior with mocks.
