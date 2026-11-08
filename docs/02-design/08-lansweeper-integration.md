# NexusAsset Cloud — Lansweeper Integration Design

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (target design) |
| Evidence level | Descriptor/design evidence; runtime integration Unverified |
| Primary evidence | `platform/backend/apps/nxa_lansweeper/**` descriptors, `prompts/04-lansweeper-integration.md`, `docs/02-design/08-lansweeper-integration.md` (prior baseline), root/architecture docs |

## Purpose
Define how NexusAsset Cloud integrates Lansweeper discovery data into tenant-scoped CMDB/ITAM records safely, idempotently, and observably.

## Integration Objectives
1. Ingest discovered IT/OT/cloud asset data into NexusAsset Cloud (`nxa_assets`/CMDB model target).
2. Preserve data integrity using idempotency, validation, mapping, and Field Ownership Matrix rules.
3. Prevent cross-tenant contamination; all sync operations run in explicit tenant context.
4. Provide operational visibility: job status, counts, conflicts, errors, retries, and audit events.
5. Handle Lansweeper API limits/failures with retries/backoff and safe recovery.

## Integration Modes (Target)
| Mode | Use case | Trigger | Status |
|---|---|---|---|
| Initial/bulk import | First-time tenant onboarding or rebuild | Manual/admin-triggered job (approved) | Target/Unverified |
| Scheduled delta sync | Regular incremental updates | Celery Beat/worker schedule | Target/Unverified |
| Webhook-driven update | Near-real-time change notification (if supported/configured) | Lansweeper webhook to verified endpoint | Target/Unverified |
| Manual resync/reconciliation | Recovery or targeted correction | Admin/operator action with scope controls | Target/Unverified |

## End-to-End Flow (Target)
| Step | Component | Action | Controls |
|---|---|---|---|
| 1 | Config/auth | Load tenant integration config and `NXA_LANSWEEPER_*` secrets from approved store | No secrets in logs/repo |
| 2 | Acquire data | Call Lansweeper GraphQL/API or receive verified webhook payload | Auth, TLS, timeout, rate-limit awareness |
| 3 | Normalize | Map external payload to internal integration DTO/domain representation | Validation, schema/version handling |
| 4 | Resolve identity | Match external asset identity to NexusAsset asset record | Deterministic keys, duplicate prevention |
| 5 | Apply ownership rules | Evaluate Field Ownership Matrix before writes | Protect business-owned/manual fields |
| 6 | Persist | Create/update/conflict via application use case and repository | Tenant scope, transactions, idempotency |
| 7 | Audit/observe | Emit audit event, metrics, structured logs, job status | Correlation IDs, redaction |
| 8 | Handle failure | Retry transient errors; quarantine/report permanent errors | Backoff, dead-letter/error status, operator visibility |

## Field Ownership Matrix (Required Target Artifact)
| Field category | Typical owner | Sync behavior | Status |
|---|---|---|---|
| Discovery facts (serial, hardware/network identifiers, discovered attributes) | Lansweeper/discovery | Upsert by approved mapping when confidence/rules allow | Target/Unverified |
| Business/governance fields (owner, cost center, contract linkage, approval state) | NexusAsset business users/processes | Do not overwrite automatically; preserve or route to conflict workflow | Target/Unverified |
| Reconciled fields | Policy-defined | Apply merge/conflict rules; record provenance | Target/Unverified |

*A concrete matrix (field name → owner → source precedence → conflict action → audit requirement) must be created and tested during implementation.*

## Idempotency and Conflict Handling
| Concern | Target control |
|---|---|
| Duplicate webhook/event delivery | Dedupe key (event ID/hash/tenant+external ID+timestamp window as designed), safe reprocessing |
| Retry after partial failure | Idempotent upsert and job checkpoint/cursor strategy |
| Concurrent syncs | Tenant/scoped locking or optimistic concurrency strategy (design choice to verify) |
| External/internal mismatch | Conflict record/status, operator review path, audit trail |
| Unsafe overwrite | Field Ownership Matrix enforcement in application/domain policy |

## Security Requirements
- Lansweeper credentials and webhook secrets use `NXA_*` and secret stores.
- Webhook endpoints verify authenticity before processing.
- Tenant mapping is authoritative server-side configuration; do not accept tenant override from untrusted payload without verified mapping rules.
- Payloads and logs are redacted for sensitive values.

## Observability Requirements (Target)
| Signal | Minimum expectation |
|---|---|
| Job status | queued/running/succeeded/failed/partial/conflict with timestamps |
| Metrics | records fetched/created/updated/skipped/conflicted/failed, duration, retry count, API/rate-limit errors |
| Logs | correlation/job/tenant IDs; no secrets; actionable error context |
| Audit | sync started/completed, config changes, privileged manual resync, significant conflicts |

## Failure Modes and Recovery
| Failure | Expected behavior | Status |
|---|---|---|
| Lansweeper API unavailable/timeout | Retry transient failures with backoff; mark job degraded/failed per policy | Target/Unverified |
| Rate limit exceeded | Respect limits, pause/backoff, reschedule | Target/Unverified |
| Invalid payload/schema change | Validate, reject/quarantine bad records, alert, preserve job diagnostics | Target/Unverified |
| Duplicate delivery | Dedupe/idempotent no-op or safe merge | Target/Unverified |
| DB constraint/conflict | Controlled error handling; no partial tenant corruption; audit/report | Target/Unverified |

## Related Files
- `06-api-contract.md`
- `09-multi-tenancy.md`
- `../07-security/01-threat-model.md`
- `../07-security/SECURITY_CONTROLS_MATRIX.md`
- `../04-testing/README.md`
- `../../prompts/04-lansweeper-integration.md`
