# NexusAsset Cloud — Webhook Contracts and Verification

## Status
Target/Unverified.

## Requirements (Target)
| Concern | Requirement |
|---|---|
| Verification | Validate signature/secret/timestamp before processing |
| Replay protection | Dedupe/idempotency window or event IDs |
| Tenant mapping | Resolve tenant from verified configuration/mapping; reject ambiguous payloads |
| Processing | Async-safe handling, retries, failure visibility |
| Audit/observability | Record receipt/result without sensitive payload leakage |
| Contract | Document payload schema, headers, responses, retry expectations |

## Lansweeper-Specific
Coordinate with `docs/02-design/08-lansweeper-integration.md` and `nxa_lansweeper` implementation prompts.

## Evidence Needed
OpenAPI/webhook docs, implementation, verification tests, and runbooks.
