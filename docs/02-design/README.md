# NexusAsset Cloud — Architecture and System Design

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (Phase B/C) |
| Evidence level | Design/ADR/descriptor evidence; most runtime flows are Target/Unverified |
| Primary evidence | `docs/02-design/*`, `docs/12-adr/*`, `prompts/*`, `platform/**/*.description.md` |

## Purpose
Define the target architecture for NexusAsset Cloud: boundaries, components, data, contracts, integration patterns, tenancy, events, and key flows. Design documentation must be detailed enough for implementation and verification, while clearly labeling what is not yet implemented.

## Architectural Principles
1. **Clean Architecture:** dependency direction inward (`Interfaces -> Application -> Domain`); infrastructure implements ports/adapters.
2. **DDD:** bounded contexts (`nxa_*`) own business language and rules.
3. **OpenAPI-first:** API contract is the source of truth (target), with codegen/contract checks.
4. **BFF for web where appropriate:** protect tokens, aggregate calls, and tailor responses.
5. **Multi-tenancy by design:** tenant context + authorization + PostgreSQL RLS target.
6. **Idempotent integrations:** Lansweeper and async operations must tolerate retries/replay safely.
7. **Observability and auditability:** significant actions produce diagnosable telemetry and audit evidence.

## Current State vs Target State
| Area | Current evidence | Target state |
|---|---|---|
| Code/runtime | Mostly `*.description.md` and structure | Executable backend/frontend/worker/mobile/infrastructure |
| OpenAPI | Descriptor/skeleton evidence | Complete OpenAPI 3.1 with endpoint catalog and codegen |
| Tenancy | Design intent | Enforced tenant checks + RLS policies + negative tests |
| Lansweeper sync | Design/prompt intent | Verified full/delta/bulk/webhook flows with idempotency and ownership matrix |
| Operations | Docs/prompts | Verified local runtime, CI gates, telemetry, runbooks |

## Contents
| File | Description | Expansion note |
|---|---|---|
| `01-c4-context.md` | C4 context and boundaries | Add external actors/systems and trust boundaries. |
| `02-c4-container.md` | Containers/runtime topology | Map `nxa-*` services, data stores, edge. |
| `03-c4-component.md` | Key service components | Detail bounded-context internals and adapters. |
| `04-clean-architecture.md` | Clean Architecture/dependency rules | Add enforcement mechanisms and examples. |
| `05-data-model.md` | Data model/ERD/RLS/indexes/retention | Mark conceptual vs verified schema. |
| `06-api-contract.md` | OpenAPI contract and API standards | Align with descriptor and endpoint catalog. |
| `07-bff-pattern.md` | BFF pattern | Define responsibilities, auth, aggregation, errors. |
| `08-lansweeper-integration.md` | Lansweeper integration | Deepen lifecycle, mapping, ownership, idempotency, failures. |
| `09-multi-tenancy.md` | Multi-tenancy strategy | Deepen tenant resolution, RLS, tests, admin scenarios. |
| `10-event-driven.md` | Domain events/async processing | Define event catalog, delivery semantics, retries. |
| `11-sequence-diagrams.md` | Key sequence diagrams | Add auth, asset CRUD, sync, webhook, failure flows. |

## Related Files
- `../12-adr/README.md`
- `../03-implementation/README.md`
- `../07-security/README.md`
- `../09-api/README.md`
