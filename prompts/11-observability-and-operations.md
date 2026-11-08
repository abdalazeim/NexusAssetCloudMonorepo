# Phase 11 — Observability and Operations

## Objective
Implement NexusAsset Cloud observability and operational readiness: metrics, logs, traces, dashboards, alerts, health checks, runbooks, and backup/DR hooks.

## Preconditions
- Phases 01–10 provide runnable services and secured operational surfaces.

## Required Context
- `infrastructure/monitoring/`
- Backend/frontend/worker logging and health endpoints
- `06-operations/` docs
- Docker/Traefik/Jenkins runtime configs

## Deliverables
1. **Telemetry instrumentation**
   - Structured logging with correlation/request/tenant context where safe.
   - Metrics for HTTP, DB, cache, Celery tasks, Lansweeper sync jobs, errors, and saturation.
   - OpenTelemetry traces across API/BFF/worker/critical integration paths as feasible.
2. **Monitoring stack configuration**
   - Prometheus scrape/config, Grafana datasource/dashboard provisioning, Loki/log pipeline, and alerting rules/Alertmanager hooks as applicable.
   - Service-level indicators/objectives for availability, latency, sync freshness, and job failure rates.
3. **Operations**
   - Liveness/readiness/startup probes and health endpoints for `nxa-backend`, `nxa-frontend`, `nxa-worker`-related checks, DB/Redis dependencies.
   - Runbooks: deploy, rollback, incident response, Lansweeper sync failure, queue backlog, DB migration, certificate/TLS issues, backup/restore verification.
   - Backup/DR procedures for PostgreSQL, Redis state where relevant, configs, and secrets references; RTO/RPO targets documented.
4. **Dashboards and alerts**
   - Minimum dashboards: platform overview, API latency/errors, worker queues, database/cache, Lansweeper integration health.
   - Alerts must be actionable and mapped to runbooks.

## Verification
┌─ Specification Block (converted from `powershell`; scope 2 — no fenced code blocks) ─┐
│ docker compose config
│ # start monitoring profile/services if defined
│ # verify /healthz or readiness endpoints
│ # verify Prometheus targets/Grafana dashboards/Loki ingestion where runtime is available
└──────────────────────────────────────────────────────────────────────────────┘
Provide static config validation and runtime verification steps.

## Exit Criteria
- Critical services expose health and telemetry hooks.
- Monitoring configs and runbooks exist for core failure modes.
- Lansweeper sync and async worker operations are observable.
- Backup/DR expectations are documented and operationally testable.
