# NexusAsset Cloud — Data Model, ERD, RLS, Indexes, and Retention

## Current State and Evidence
Status: Expanded baseline (Phase B/C). Evidence is Descriptor/Target/Planned unless verified by executable implementation, tests, scans, or runtime output.

## Current State and Evidence
No verified schema dump/migrations/runtime ERD in current repository state. This document defines target data-modeling requirements.

## Data Modeling Requirements (Target)
| Requirement | Detail |
|---|---|
| Tenant scope | Tenant-owned tables include `tenant_id` (or equivalent) and support RLS where applicable |
| Identity/integrity | Primary keys, foreign keys, unique constraints, and nullability reflect domain invariants |
| Provenance | Discovery-sourced records retain source identifiers/provenance for idempotency and reconciliation |
| Auditability | Sensitive changes have audit/event strategy (`nxa_audit` or approved mechanism) |
| Performance | Indexes for tenant filters, common lookups, foreign keys, sync cursors/external IDs |
| Retention | Retention/archive/delete policies defined per data class and compliance needs |

## Conceptual Entities (Target)
Tenants, users/memberships/roles, assets, asset relationships, discovery source records (Lansweeper), sync jobs/cursors/conflicts, notifications, billing/commercial records, audit events. Exact schemas must be verified from migrations/models when implemented.

## RLS Notes
PostgreSQL RLS is defense-in-depth, not a replacement for application authorization. Policies, session/context mechanism, admin bypass rules, and tests must be documented and verified.

## Required Evidence for Completion
Approved ERD from implemented schema, migration history, RLS policies, index review, and retention/runbook documentation.
