# NexusAsset Cloud — Clean Architecture and Dependency Rules

## Current State and Evidence
Status: Expanded baseline (Phase B/C). Evidence is Descriptor/Target/Planned unless verified by executable implementation, tests, scans, or runtime output.

## Purpose
Ensure NexusAsset Cloud business rules remain testable and independent from frameworks, UI, databases, and external integrations.

## Dependency Rule
| Layer | May depend on | Must not depend on |
|---|---|---|
| Domain | Standard language/library primitives as approved; domain-only abstractions | Django/DRF/ORM/HTTP/Celery/UI/external SDKs |
| Application | Domain, ports/interfaces, DTOs | Concrete ORM/web/framework details |
| Infrastructure | Application ports, domain models as needed for mapping, frameworks/SDKs | Owning business rules |
| Interfaces | Application/use cases, DTOs/schemas | Direct domain persistence shortcuts that bypass application rules |

## Enforcement (Target)
- Import-linter/architecture tests in CI.
- Code review checklist for new modules/endpoints/jobs.
- ADR required for approved exceptions.

## NexusAsset Cloud Application
- All backend bounded contexts follow `domain/application/infrastructure/interfaces` separation.
- Lansweeper API client is infrastructure; sync decision logic (mapping precedence, field ownership, conflict policy) belongs in domain/application policy.
- Tenant and authorization checks are enforced server-side in application/interface/data layers.
