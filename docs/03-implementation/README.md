# NexusAsset Cloud — Project Structure, Standards, and Engineering Patterns

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (Phase B/C) |
| Evidence level | Structure/standards documented; implementation patterns are Target/Descriptor unless code is verified |
| Primary evidence | `docs/03-implementation/*`, `platform/**` descriptors, `project-meta/*` descriptors, `prompts/*` |

## Purpose
Translate NexusAsset Cloud architecture into enforceable engineering standards: repository structure, naming, layer responsibilities, coding patterns, migrations, error handling, transactions, and feature-flag governance.

## Current State vs Target State
The repository currently contains target structures and descriptor artifacts under `platform/`. Engineering standards in this module apply to future/ongoing implementation and must be used when regenerating executable artifacts from descriptors.

## Implementation Rules (Mandatory)
- Keep Domain independent from frameworks and infrastructure concerns.
- Use `nxa_*` bounded-context packages under the backend app structure.
- Use canonical `nxa-*` service/container names in infrastructure.
- Treat OpenAPI 3.1 as the source of truth (target); current descriptor: `platform/api/contracts/openapi.yaml.description.md` (current descriptor; target contract path: `api/contracts/openapi` + YAML).
- Use `NXA_*` for environment/secret keys; never commit real secrets.
- Enforce tenant isolation and dependency rules in code review and CI (target gates).

## Bounded-Context Internal Pattern (Target)
| Layer | Contains | Must not contain |
|---|---|---|
| `domain` | Entities, value objects, domain services, domain events/rules | Django/DRF/ORM imports, HTTP/Redis/Celery specifics |
| `application` | Use cases, DTOs, ports, orchestration | Framework persistence details, transport-specific concerns |
| `infrastructure` | ORM models/migrations adapters, repositories, external clients, mappers | Business rule ownership |
| `interfaces` | API views/serializers/controllers, task handlers, command adapters | Core business decisions that belong in domain/application |

## Contents
| File | Description | Expansion note |
|---|---|---|
| `01-project-structure.md` | Canonical monorepo and backend layout | Keep reconciled with current `platform/` + target paths. |
| `02-naming-conventions.md` | Naming standards | Add examples and anti-patterns. |
| `03-solid-principles.md` | SOLID application | Map to NexusAsset Cloud examples/status labels. |
| `04-clean-code.md` | Readability/maintainability | Add review checklist. |
| `05-dependency-rule.md` | Dependency direction and enforcement | Add import-linter/CI target checks. |
| `06-use-case-pattern.md` | Use-case pattern | Add input/output/error/transaction expectations. |
| `07-repository-pattern.md` | Ports/repositories/persistence boundaries | Add tenant and mapping rules. |
| `08-mappers.md` | Domain/DTO/ORM mapping | Add field ownership/sensitive-field rules. |
| `09-migrations-strategy.md` | Expand/Contract migrations | Add RLS/multi-tenant migration cautions. |
| `10-error-handling.md` | Error mapping | Add domain/application/API/observability mapping. |
| `11-transactions.md` | Unit of Work/transaction boundaries | Add idempotency and async boundaries. |
| `12-feature-flags.md` | Feature flag lifecycle | Add rollout, cleanup, and audit considerations. |

## Related Files
- `../02-design/04-clean-architecture.md`
- `../04-testing/09-architecture-testing.md`
- `../documentation-audit/STATUS_TAXONOMY.md`
