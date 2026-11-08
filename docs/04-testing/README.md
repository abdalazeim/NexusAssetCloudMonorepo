# NexusAsset Cloud — Test Strategy and Quality Verification

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Baseline documentation expanded (Phase B/C in progress) |
| Evidence level | Target/Planned; no test execution reports or verified coverage in current repository state |
| Primary evidence | `prompts/07-testing-and-quality-gates.md`, `docs/04-testing/*`, `platform/tests/*` descriptors, quality-gate references in prompts/project-meta |
| Constraint | Commands and tools listed below are target verification commands; mark as Unverified until executed in an implemented environment. |

## Purpose
Define the NexusAsset Cloud quality strategy: test pyramid, test types, environments, data management, CI quality gates, evidence requirements, and Definition of Done.

## Quality Principles
1. Test behavior and contracts, not only implementation details.
2. Protect critical invariants: tenant isolation, authorization, Lansweeper idempotency/field ownership, API contract compatibility.
3. Prefer deterministic tests with isolated data/fixtures and mocked external systems (Lansweeper, SMTP/providers, third-party APIs).
4. Make failures actionable with logs, correlation IDs, and reproducible inputs.
5. No claim of coverage, pass rate, or performance is complete without evidence.

## Test Pyramid and Scope (Target)
| Layer | Focus | Typical tools (target) | Evidence required |
|---|---|---|---|
| Unit | Domain/application logic, validation, mapping, policies | pytest, Vitest/Jest as applicable | Test files + pass results |
| Integration | DB/repository/adapters, API + persistence, Redis/Celery boundaries | pytest + test DB/services | Environment + pass results |
| Contract | OpenAPI compatibility, codegen drift, schema validation | OpenAPI validators, contract tests | Contract report / CI logs |
| Architecture | Clean Architecture dependency rule, module boundaries | import-linter/AST tests | Rule config + pass results |
| E2E/smoke | Critical user/API journeys | Playwright/API smoke | Scenario list + results |
| Security | AuthZ/tenant isolation negative tests, scans | security tests + SAST/DAST/secret/dependency scanners | Reports + triage |
| Performance/load | Latency/throughput/capacity for key flows | k6/Locust or equivalent | Baseline metrics + thresholds |
| Accessibility | Web UI WCAG-oriented checks (when UI implemented) | automated/manual checks | Findings + remediation |

## Critical Test Scenarios (Minimum for MVP)
| ID | Scenario | Type | Status |
|---|---|---|---|
| T-TENANT-001 | User from tenant A cannot read/update/delete tenant B assets/data | Integration/security | Target/Unverified |
| T-AUTH-001 | Unauthenticated/expired/invalid token requests are rejected appropriately | API/integration | Target/Unverified |
| T-CONTRACT-001 | Implemented API responses match OpenAPI 3.1 contract | Contract | Target/Unverified |
| T-LS-001 | Lansweeper sync is idempotent (same payload/event/cursor does not duplicate records) | Unit/integration | Target/Unverified |
| T-LS-002 | Field Ownership Matrix prevents unsafe overwrite of business-owned fields | Unit/integration | Target/Unverified |
| T-AUDIT-001 | Significant sync/security/business actions create audit records | Integration | Target/Unverified |
| T-HEALTH-001 | Health/readiness endpoints report service/dependency status correctly | Smoke/integration | Target/Unverified |

## Environments and Test Data
| Environment | Purpose | Data policy |
|---|---|---|
| Local dev | Developer feedback loop | Synthetic/non-production data only |
| CI | Automated gates on PR/main | Ephemeral fixtures/mocks; no production secrets |
| Staging | Integration/UAT/DAST/perf baseline | Sanitized representative data; controlled access |
| Production | Monitoring/operability only | No destructive test data injection |

## Quality Gates (Target)
- Lint/format.
- Type checks.
- Architecture dependency checks.
- Unit/integration tests.
- Contract/OpenAPI validation and codegen consistency.
- Security checks (secrets/dependency/SAST; DAST in staging where configured).
- Build/package verification.

## Contents
| File | Description | Audit note |
|---|---|---|
| `01-test-strategy.md` | Test pyramid and strategy | Expand with evidence requirements and MVP gates. |
| `02-unit-testing.md` | Unit testing standards | Expand with patterns for domain/application tests. |
| `03-integration-testing.md` | Integration testing standards | Expand with DB/Redis/Celery/API setup and isolation. |
| `04-contract-testing.md` | OpenAPI contract testing | Expand with validation/codegen drift workflow. |
| `05-e2e-testing.md` | E2E testing | Expand with critical journeys and smoke scope. |
| `06-load-testing.md` | Load/performance testing | Expand with scenarios, metrics, thresholds. |
| `07-security-testing.md` | Security testing controls | Expand with negative tests and scanner triage. |
| `08-a11y-testing.md` | Accessibility testing | Expand for web UI implementation phase. |
| `09-architecture-testing.md` | Architecture/dependency testing | Expand with Clean Architecture enforcement. |
| `10-definition-of-done.md` | Definition of Done | Expand with status/evidence checklist. |

## Related Files
- `../03-implementation/README.md`
- `../07-security/README.md`
- `../09-api/README.md`
- `../../prompts/07-testing-and-quality-gates.md`
