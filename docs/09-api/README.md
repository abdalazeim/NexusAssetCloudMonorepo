# NexusAsset Cloud — API Contract and Integration Standards

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Baseline documentation expanded (Phase B/C in progress) |
| Evidence level | Implemented (Descriptor Evidence) for contract intent; runtime endpoints largely Target/Unverified |
| Primary evidence | `platform/api/contracts/openapi.yaml.description.md`, `prompts/03-api-openapi-and-contracts.md`, `docs/09-api/*`, `docs/02-design/06-api-contract.md` |
| Known limitation | The OpenAPI descriptor currently evidences a skeleton contract (e.g., `GET /healthz` target) — not a complete NexusAsset Cloud API surface. |

## Purpose
This module defines NexusAsset Cloud API governance: OpenAPI 3.1 source-of-truth rules, authentication/authorization expectations, versioning, pagination/filtering/sorting, error envelopes, rate limiting, webhooks, and SDK/code generation practices.

## OpenAPI-First Contract Rules
1. OpenAPI 3.1 is the authoritative contract for backend, BFF, frontend, mobile, and generated clients.
2. Contract changes must be reviewed for backward compatibility; breaking changes require explicit versioning/deprecation.
3. Every implemented endpoint must have: operation ID, summary/description, auth requirements, parameters, request/response schemas, and error responses.
4. Generated clients/types must be reproducible from the contract; CI should detect drift (Target).
5. Examples must not contain real secrets, tokens, or tenant production data.

## Endpoint Documentation Standard (Required When Implemented)
| Field | Requirement |
|---|---|
| Method & URL | Exact HTTP method and path |
| Purpose | Business/technical purpose and bounded context |
| AuthN/AuthZ | JWT/BFF/session requirements, roles/permissions, tenant scope |
| Headers | Required/optional headers, idempotency keys where needed |
| Path/query/body | Types, validation, defaults, constraints |
| Responses | Success schemas, status codes, pagination envelope |
| Errors | Error envelope, codes, retryability guidance |
| Examples | Realistic sanitized request/response examples |
| Status | Implemented (Verified) / Descriptor / Target / Planned / Unverified |

## Core API Domains (Target)
| Domain | Bounded context | Status |
|---|---|---|
| Health/readiness | Cross-cutting | Target / Descriptor Evidence |
| Authentication/bootstrap | `nxa_accounts` | Target / Unverified |
| Tenant context | `nxa_tenants` | Target / Unverified |
| Assets/CMDB | `nxa_assets` | Target / Unverified |
| Lansweeper integration/sync status | `nxa_lansweeper` | Target / Unverified |
| Audit events | `nxa_audit` | Target / Unverified |
| Notifications | `nxa_notifications` | Planned / Target |
| Billing/commercial | `nxa_billing` | Planned / Target |

## Error and Pagination Standards (Target)
- Use a consistent error envelope (code, message, details, correlation/request ID as appropriate) without leaking sensitive internals.
- Support pagination/filter/sort for list endpoints with documented limits and deterministic ordering.
- Use idempotency keys for operations where retries may create duplicates (especially integrations/async triggers).

## Contents
| File | Description | Audit note |
|---|---|---|
| `01-openapi-spec.md` | OpenAPI governance | Expand with contract lifecycle, review checklist, codegen, versioning hooks. |
| `02-authentication.md` | JWT/session/BFF usage | Expand with token placement, scopes/claims, error cases, status labels. |
| `03-versioning.md` | API versioning/deprecation | Expand with compatibility policy and migration workflow. |
| `04-pagination.md` | Pagination/filtering/sorting | Expand with parameter names, limits, cursor/page rules. |
| `05-error-codes.md` | Error envelope/codes | Define canonical catalog and HTTP mapping. |
| `06-rate-limiting.md` | Rate limits/throttling | Expand per edge/user/device/IP strategy and response behavior. |
| `07-webhooks.md` | Webhook contracts/verification | Expand Lansweeper/external webhook verification, replay, idempotency. |
| `08-sdks.md` | Generated clients/SDKs | Expand generation workflow and package ownership (`@nexusasset/*`). |

## Required Phase B/C Additions
- Endpoint catalog table generated/reconciled from the approved OpenAPI contract.
- BFF/mobile contract matrix (what goes through BFF vs direct API, if applicable).
- Example request/response set with sanitized data and explicit status labels.

## Related Files
- `../02-design/06-api-contract.md`
- `../02-design/07-bff-pattern.md`
- `../03-implementation/README.md`
- `../07-security/04-authentication.md`
- `../../platform/api/contracts/openapi.yaml.description.md`
