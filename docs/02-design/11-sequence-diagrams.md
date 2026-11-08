# NexusAsset Cloud — Key Sequence Diagrams

## Current State and Evidence
Status: Expanded baseline (Phase B/C). Evidence is Descriptor/Target/Planned unless verified by executable implementation, tests, scans, or runtime output.

## Status
Target documentation. Use ASCII/text diagrams (Mermaid deferred unless repository policy exception is approved).

## Required Flows to Document
| Flow | Key checks |
|---|---|
| User login/authenticated request | TLS, auth validation, tenant resolution, authorization, error handling |
| BFF to backend API call | Session/token handling, contract mapping, timeout/retry behavior |
| Asset list/detail query | Tenant filter, pagination, authorization, safe error response |
| Lansweeper scheduled delta sync | Config/secrets, fetch, normalize, ownership/conflict, idempotent upsert, audit/metrics |
| Lansweeper webhook | Verification, dedupe, tenant mapping, async processing, response semantics |
| Sync failure/retry | Backoff, job status, operator visibility, no data corruption |

## Diagram Standard
Each diagram must state actors/components, preconditions, main path, failure paths, and status (Target/Verified).

## Example (Text Sequence — Target)
User -> BFF: request asset list
BFF -> API: authenticated request with server-resolved context
API -> AuthZ/Tenant: validate principal and tenant scope
API -> Application: query assets use case
Application -> Repository: tenant-scoped query
Repository -> PostgreSQL: filtered read (RLS defense-in-depth target)
PostgreSQL -> Repository -> Application -> API -> BFF -> User: paginated result or safe error
