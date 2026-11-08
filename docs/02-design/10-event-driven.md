# NexusAsset Cloud — Domain Events and Asynchronous Processing

## Current State and Evidence
Status: Expanded baseline (Phase B/C). Evidence is Descriptor/Target/Planned unless verified by executable implementation, tests, scans, or runtime output.

## Status
Target/Unverified.

## Purpose
Use asynchronous processing for long-running, retryable, or integration-heavy work (especially Lansweeper sync and notifications) while preserving idempotency, tenant scope, and auditability.

## Event/Job Requirements (Target)
| Requirement | Detail |
|---|---|
| Tenant context | Events/jobs carry validated tenant scope; no implicit cross-tenant processing |
| Idempotency | Handlers tolerate retries/duplicate delivery |
| Schema/versioning | Event payloads versioned or backward-compatible as adopted |
| Observability | Correlation IDs, job status, metrics, structured logs |
| Failure handling | Retry policy, backoff, dead-letter/quarantine strategy as adopted |
| Audit | Significant state changes emit audit evidence |

## Candidate Flows
Asset discovered/synced, sync conflict detected, audit event recorded, notification requested/sent, billing linkage updated (as scoped).

## Evidence Needed
Task/event implementation, broker config, retry tests, idempotency tests, and runbooks.
