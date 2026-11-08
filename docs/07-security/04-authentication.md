# NexusAsset Cloud — Authentication Flows and Controls

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (target design) |
| Evidence level | Target/Unverified; no verified runtime auth implementation claimed |
| Primary evidence | `docs/09-api/02-authentication.md`, `docs/02-design/07-bff-pattern.md`, `prompts/03-api-openapi-and-contracts.md`, `project-meta/.env.example.description.md` |

## Purpose
Define the target authentication architecture for NexusAsset Cloud web/BFF, API, and mobile clients, including token lifecycle, failure handling, and verification requirements.

## Target Authentication Model
| Component | Responsibility | Status |
|---|---|---|
| Identity provider / account domain (`nxa_accounts`) | User identity, credential validation, account status, roles/claims source | Target |
| BFF (`nxa-frontend` boundary) | Browser-safe auth interaction, token/session mediation, aggregation | Target |
| API (`nxa-backend`) | Token validation, tenant/permission enforcement, protected resources | Target |
| Mobile clients | Secure token storage, refresh, authorized API calls | Target |

## Login Lifecycle (Target — to verify during implementation)
| Step | Description | Security control |
|---|---|---|
| 1 | Client submits credentials/login request through approved channel | TLS, rate limiting, input validation |
| 2 | System validates user existence/status and credential verifier | Constant-time verification where applicable, generic failure responses |
| 3 | System resolves authorized tenant/role context as designed | No client-trusted tenant authorization |
| 4 | System issues access token (and refresh/session mechanism if approved) | Signed token, minimal claims, expiry |
| 5 | Client uses token/session for subsequent requests | Secure storage/transport; BFF avoids unnecessary browser token exposure |
| 6 | API validates signature, expiry, claims, tenant/permission requirements | Deny on invalid/expired/missing auth |
| 7 | Refresh/reauth occurs per policy | Rotation/revocation strategy as implemented |
| 8 | Logout/revocation invalidates usable access according to design | Audit significant auth events |

## Token and Session Considerations (Target)
- Access token TTL and refresh TTL are configuration-driven (`NXA_*` as described in descriptors); values must be validated for security/usability.
- Claims must be minimal and must not include secrets.
- Tenant/authorization decisions must be enforced server-side; token claims may inform but not replace authorization checks.
- Revocation/blacklist/rotation behavior is Unverified until implemented.

## Failure Handling
| Case | Expected behavior | Status |
|---|---|---|
| Invalid credentials | Generic unauthorized response; no user enumeration leakage | Target |
| Disabled/locked user | Deny access per policy; audit as appropriate | Target |
| Expired token | Unauthorized/reauth flow; clear client handling | Target |
| Invalid signature/claims | Reject request; log safely | Target |
| Missing tenant/permission | Forbidden/unauthorized by policy; no cross-tenant fallback | Target |

## Verification Requirements
- Unit/integration tests for token validation and auth middleware/guards.
- Negative tests for expired/invalid/missing tokens.
- Tenant/permission enforcement tests (coordinate with authorization docs).
- Security review of BFF cookie/session/CSRF/CORS behavior when implemented.

## Related Files
- `05-authorization.md`
- `../02-design/07-bff-pattern.md`
- `../09-api/02-authentication.md`
