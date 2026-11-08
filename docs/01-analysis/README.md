# NexusAsset Cloud — Requirements, Domain, and Risk Analysis

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (Phase B/C) |
| Evidence level | Analysis intent documented; requirements/acceptance traceability is incomplete and largely Target/Unverified |
| Primary evidence | `docs/01-analysis/*`, `docs/02-design/*`, `prompts/*`, root evaluation files |

## Purpose
Capture who NexusAsset Cloud serves, what problems it must solve, how the domain is modeled, and which risks constrain delivery. This module is the bridge between business intent and technical design.

## Analysis Method (Required)
1. Identify stakeholders, personas, and jobs-to-be-done.
2. Define functional requirements (FR), non-functional requirements (NFR), security requirements (SEC), operational requirements (OPS), and integration requirements (INT).
3. Model the domain using bounded contexts, aggregates/entities/value objects, events, and context relationships.
4. Specify acceptance criteria in testable form.
5. Maintain traceability to design, OpenAPI, implementation evidence, and tests (see `../documentation-audit/REQUIREMENTS_TRACEABILITY_MATRIX.md`).

## Bounded Contexts (Canonical)
| Context | Responsibility (target) | Key risks |
|---|---|---|
| `nxa_core` | Shared primitives/cross-cutting building blocks | Over-coupling, unclear ownership |
| `nxa_accounts` | Identity, users, auth-related capabilities | Credential/token vulnerabilities |
| `nxa_tenants` | Tenant lifecycle/context/isolation controls | Cross-tenant access |
| `nxa_assets` | Asset/CMDB inventory and relationships | Data model drift, integrity issues |
| `nxa_lansweeper` | Discovery integration and synchronization | Duplicates, unsafe overwrites, replay/rate limits |
| `nxa_billing` | Commercial records | Incorrect linkage/authorization |
| `nxa_notifications` | Event-driven notifications | Delivery failures, noise |
| `nxa_audit` | Audit events and evidence | Missing/incomplete audit trails |

## Requirements Quality Rules
| Rule | Requirement |
|---|---|
| Testable | Each MVP requirement has acceptance criteria and verification method. |
| Scoped | Requirement states tenant, role, data, and error behavior where relevant. |
| Traceable | Requirement maps to context, API/contract, implementation, and test evidence. |
| Status-labeled | Implemented/Descriptor/Target/Planned/Unverified is explicit. |
| No speculation | Unconfirmed behavior is not documented as fact. |

## Contents
| File | Description | Expansion note |
|---|---|---|
| `01-stakeholders.md` | Stakeholders and responsibilities | Add decision rights, RACI-style ownership as needed. |
| `02-personas.md` | User personas and goals | Add pain points, success criteria, access needs. |
| `03-functional-requirements.md` | Functional requirements | Convert to FR IDs with acceptance criteria. |
| `04-non-functional-requirements.md` | Non-functional requirements | Add measurable targets and verification method. |
| `05-use-cases.md` | Use cases by bounded context | Add pre/postconditions, failures, security. |
| `06-user-stories.md` | User stories and acceptance criteria | Use Given/When/Then and status labels. |
| `07-domain-model.md` | Domain entities/VOs/aggregates/events | Align with Clean Architecture and `nxa_*` contexts. |
| `08-bounded-contexts.md` | Bounded contexts and responsibilities | Add ownership and interface contracts. |
| `09-context-map.md` | Context map/integration relationships | Add upstream/downstream and failure modes. |
| `10-risk-register.md` | Architectural and delivery risks | Add likelihood/impact/mitigation/owner/status. |

## Related Files
- `../documentation-audit/REQUIREMENTS_TRACEABILITY_MATRIX.md`
- `../02-design/README.md`
- `../07-security/README.md`
- `../../prompts/README.md`
