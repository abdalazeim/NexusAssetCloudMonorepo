# NexusAsset Cloud — OpenAPI 3.1 Specification Governance

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Baseline governance spec |
| Evidence level | Descriptor Evidence (skeleton contract) + Target |
| Primary evidence | `platform/api/contracts/openapi.yaml.description.md`, `prompts/03-api-openapi-and-contracts.md` |

## Contract Authority
- OpenAPI 3.1 is the single source of truth for NexusAsset Cloud API behavior (target).
- Current descriptor evidences a minimal/skeleton contract (`GET /healthz` target evidence) — do not present a complete implemented API surface.

## Required Contract Content (Target)
- Metadata: title, version, servers (`app`/`api` domains as appropriate), tags.
- Security schemes (JWT/BFF-aligned as designed).
- Reusable schemas: common identifiers, pagination, errors, tenant-safe resource models.
- Paths/operations with operation IDs, parameters, request/response schemas, status codes, examples, and auth requirements.
- Webhook/callback specs where used.

## Change Management
| Change type | Requirement |
|---|---|
| Backward-compatible addition | Allowed with review; update clients/codegen |
| Behavioral change | Document clearly; add tests and migration notes |
| Breaking change | Versioning/deprecation plan, consumer communication, rollout/rollback plan |

## Validation Gates (Target)
- OpenAPI schema validation.
- Contract tests against implementation.
- Codegen drift detection.
- Security review for auth/tenant/error exposure.
