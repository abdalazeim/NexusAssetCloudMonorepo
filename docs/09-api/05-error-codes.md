# NexusAsset Cloud — Error Envelope and Error Codes

## Status
Target/Unverified.

## Error Envelope (Target)
All non-success API responses should use a consistent, documented structure (example fields): `code`, `message`, `details` (safe/validated), `request_id`/correlation ID where available. Do not expose stack traces, secrets, SQL, or sensitive internals.

## HTTP Mapping (Target)
| HTTP | Typical use |
|---|---|
| 400 | Validation/client input error |
| 401 | Authentication required/invalid |
| 403 | Authenticated but not authorized / tenant access denied as policy defines |
| 404 | Resource not found (avoid tenant existence leakage) |
| 409 | Conflict/idempotency/state conflict |
| 422 | Semantic validation (if adopted) |
| 429 | Rate limited |
| 5xx | Server/integration failure; safe message + correlation ID |

## Evidence Needed
Canonical error catalog in OpenAPI, implementation, and contract/security tests.
