# NexusAsset Cloud — Runtime Operations and Reliability

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Baseline documentation expanded (Phase B/C in progress) |
| Evidence level | Target/Descriptor Evidence; no verified runtime telemetry, dashboards, or incident records |
| Primary evidence | `prompts/11-observability-and-operations.md`, `docs/06-operations/*`, deployment/security/testing docs |
| Constraint | Operational runbooks must not claim live endpoints, dashboards, or backup jobs as verified until executed and evidenced. |

## Purpose
Define NexusAsset Cloud operational standards for health, observability, alerting, incidents, backups, disaster recovery, and routine maintenance.

## Operational Objectives
1. Detect service degradation and integration failures quickly.
2. Protect tenant data integrity and availability.
3. Provide actionable diagnostics (logs/metrics/traces) without exposing secrets.
4. Restore service within defined RTO/RPO targets (to be approved).
5. Maintain auditable operational actions for sensitive changes.

## Observability Model (Target)
| Signal | Purpose | Key NexusAsset Cloud focus |
|---|---|---|
| Logs | Event-level diagnostics and audit support | request/correlation IDs, sync job outcomes, auth failures, errors; no secrets |
| Metrics | Quantitative health/performance | HTTP latency/errors, queue depth, worker success/failure, DB/Redis health, sync lag/conflicts |
| Traces | End-to-end request/job flow | BFF/API/worker/Lansweeper integration paths |
| Alerts | Actionable notifications | SLO breaches, error spikes, queue backlog, sync failures, DB saturation |

## Core Runbook Areas (Required)
| Area | Minimum content | Status |
|---|---|---|
| Service startup/health | checks, dependencies, expected healthy signals | Target |
| Lansweeper sync failure | symptoms, diagnosis, retry/idempotency checks, escalation | Target |
| Webhook replay/abuse | verification, dedup/idempotency, rate controls | Target |
| Queue backlog | worker scaling, poison messages/retries, dependency checks | Target |
| Tenant access anomaly | containment, audit/log review, RLS/app checks | Target |
| Database migration issue | prechecks, rollback/forward-fix, data integrity checks | Target |
| Backup/restore | schedule, verification, restore drill, RTO/RPO evidence | Target |
| Incident response | severity, communication, containment, RCA, corrective actions | Target |

## Backup and Disaster Recovery (Target)
| Asset | Strategy (to be implemented/verified) | Evidence required |
|---|---|---|
| PostgreSQL | Scheduled backups, restore tests, replication/PITR as approved | Backup logs, restore drill report |
| Redis | Persistence/backup strategy appropriate to workload; treat as cache/broker unless state requirements differ | Config + recovery notes |
| Config/secrets references | Versioned non-secret config; secrets in approved stores with recovery process | Procedures + access controls |
| Observability data | Retention and recovery policy per environment | Policy + config evidence |

## Contents
| File | Description | Audit note |
|---|---|---|
| `01-observability.md` | Observability strategy | Expand with telemetry standards and evidence paths. |
| `02-monitoring.md` | Monitoring/dashboards | Define dashboard list and golden signals. |
| `03-logging.md` | Structured logging | Define schema, redaction, correlation/tenant context rules. |
| `04-tracing.md` | Distributed tracing | Define span coverage and propagation. |
| `05-alerting.md` | Alerting/escalation | Define actionable alerts and ownership. |
| `06-slo-sli.md` | SLO/SLI | Define measurable targets and error budgets. |
| `07-runbooks.md` | Operational runbooks | Expand with step-by-step diagnosis/remediation. |
| `08-incident-response.md` | Incident response | Expand with severity model and communication/RCA. |
| `09-backup-dr.md` | Backup/DR | Expand with restore drills and RTO/RPO. |

## Related Files
- `../05-deployment/README.md`
- `../07-security/README.md`
- `../04-testing/06-load-testing.md`
- `../../prompts/11-observability-and-operations.md`
