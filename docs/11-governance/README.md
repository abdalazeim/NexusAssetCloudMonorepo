# NexusAsset Cloud — Engineering Process and Delivery Governance

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (Phase B/C) |
| Evidence level | Governance intent and policy descriptors exist; operational enforcement in CI/team workflows may be Unverified |
| Primary evidence | `docs/11-governance/*`, `project-meta/CONTRIBUTING.md`, `project-meta/SECURITY.md`, `prompts/*`, ADRs |

## Purpose
Define how NexusAsset Cloud changes are proposed, reviewed, approved, released, and documented. Governance ensures technical quality, security, tenant safety, and traceability.

## Governance Controls (Target)
| Control | Requirement | Status |
|---|---|---|
| Change management | Issues/tasks linked to scoped PRs/commits | Target/partially process-defined |
| Code review | Required reviewers for sensitive paths (API, security, tenancy, infra) | Target |
| ADRs | Significant architecture decisions recorded with status/context/consequences | Descriptor/design evidence present |
| Quality gates | Lint, types, architecture, tests, contract, security, build | Target/Unverified |
| Release governance | Versioning, release notes, rollback readiness, approvals | Target/Unverified |
| Documentation governance | Docs updated with behavior/contract/architecture changes | Target |

## Definition of Ready / Definition of Done (Summary)
- **Ready:** scope clear, acceptance criteria defined, dependencies known, security/tenant impact assessed, status/evidence expectations identified.
- **Done:** implementation/docs/tests/evidence updated as applicable; gates pass; risks and rollout/rollback considerations documented.

## Contents
| File | Description | Audit note |
|---|---|---|
| `01-git-workflow.md` | Branching, commits, PRs | Expand with protected-branch and review rules. |
| `02-code-review.md` | Code review standards | Add security/tenancy/API/architecture checklist. |
| `03-definition-of-ready.md` | Definition of Ready | Add measurable entry criteria. |
| `04-release-versioning.md` | SemVer and release management | Add artifact/version alignment. |
| `05-team-structure.md` | Team ownership and collaboration | Add ownership boundaries and escalation. |

## Related Files
- `../12-adr/README.md`
- `../04-testing/10-definition-of-done.md`
- `../../project-meta/CONTRIBUTING.md`
