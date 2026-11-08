# NexusAsset Cloud — Backend For Frontend (BFF) Pattern

## Current State and Evidence
Status: Expanded baseline (Phase B/C). Evidence is Descriptor/Target/Planned unless verified by executable implementation, tests, scans, or runtime output.

## Status
Target/Descriptor Evidence.

## Purpose
Provide a web-focused boundary that can aggregate backend calls, manage browser-safe auth/session concerns, reduce chattiness, and shape responses for UI needs without embedding business ownership rules that belong in backend bounded contexts.

## Responsibilities (Target)
| Responsibility | BFF | Backend |
|---|---|---|
| UI-oriented aggregation | Yes | Optional/internal services |
| Browser auth/session handling | Yes, as designed | Token/user validation and authorization authority |
| Business rules/invariants | No | Yes (domain/application) |
| Tenant authorization | Must enforce/propagate correctly | Final authority |
| Contract alignment | Must align with OpenAPI/backend contracts | Source of domain/API behavior |

## Risks
- Duplicating business logic in BFF.
- Leaking tokens or sensitive data to browser.
- Bypassing backend authorization/tenant checks.
- Untracked divergence from OpenAPI/client contracts.

## Verification
BFF auth/CSRF/CORS tests, contract tests, tenant-negative tests, and security review.
