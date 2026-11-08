# NexusAsset Cloud — Architecture and Flow Diagrams

## Current State and Evidence
Status: Expanded baseline (Phase B/C). Evidence is Descriptor/Target/Planned unless verified by executable implementation, tests, scans, or runtime output.

## Status
Expanded baseline. Diagram policy: ASCII/text/specification diagrams by default (Mermaid deferred unless repository policy exception is approved).

## Required Diagram Catalog (Target)
| Diagram | Source docs | Status |
|---|---|---|
| System context | `docs/02-design/01-c4-context.md` | Target |
| Container/runtime topology | `docs/02-design/02-c4-container.md` | Target |
| Bounded-context components | `docs/02-design/03-c4-component.md` | Target |
| Clean Architecture layers | `docs/02-design/04-clean-architecture.md` | Target |
| Conceptual ERD/data ownership | `docs/02-design/05-data-model.md` | Target/Unverified schema |
| Auth request lifecycle | `docs/07-security/04-authentication.md`, `docs/09-api/02-authentication.md` | Target |
| Tenant isolation flow | `docs/02-design/09-multi-tenancy.md` | Target |
| Lansweeper sync/webhook flow | `docs/02-design/08-lansweeper-integration.md` | Target |
| CI/CD and deployment topology | `docs/05-deployment/*` | Target |
| Incident/backup-recovery flow | `docs/06-operations/*` | Target |
