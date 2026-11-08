# NexusAsset Cloud — API Authentication

## Status
Target/Unverified. Aligns with `docs/07-security/04-authentication.md` and BFF design.

## Requirements (Target)
| Area | Expectation |
|---|---|
| Protected endpoints | Require valid authentication unless explicitly public (e.g., health as designed) |
| Token format | JWT or approved equivalent; claims minimal and non-sensitive |
| Tenant context | Server-side resolution/enforcement; never trust client-supplied tenant for authorization |
| Errors | `401`/`403` behavior consistent and non-leaky |
| BFF/browser flows | CSRF/CORS/session handling documented when implemented |
| Mobile flows | Secure storage and refresh behavior documented when implemented |

## Evidence Needed
OpenAPI security schemes, implemented middleware/guards, positive/negative auth tests, and threat-model review.
