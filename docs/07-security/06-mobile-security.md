# NexusAsset Cloud — Mobile Client Security Controls

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (target design) |
| Evidence level | Target/Unverified; mobile implementation not verified in current repository state |
| Primary evidence | `docs/08-mobile/*`, `docs/07-security/04-authentication.md`, `prompts/06-mobile-android.md` |

## Purpose
Define security controls for NexusAsset Cloud mobile clients (Android baseline, iOS target/planned where applicable).

## Controls (Target)
| Area | Control | Status |
|---|---|---|
| Transport | HTTPS/TLS for all API communication; certificate validation enabled | Target/Unverified |
| Token storage | Use secure platform storage (e.g., Android Keystore-backed mechanisms) for sensitive tokens; avoid plaintext shared prefs/files | Target/Unverified |
| Auth lifecycle | Expiry/refresh handling, secure logout, revoked-session behavior | Target/Unverified |
| Least data | Cache minimum necessary tenant/asset data; protect local storage | Target/Unverified |
| Input/output | Validate deep links/intents/inputs; handle errors without leaking sensitive details | Target/Unverified |
| Device posture | Optional MDM/attestation/root detection policies only if approved; avoid false security claims | Planned/Unverified |
| Logging | No secrets/tokens/PII in mobile logs or crash reports beyond approved policy | Target/Unverified |
| Offline mode | If implemented, encrypt/protect local cache and resolve conflicts safely; mark Unverified until implemented | Planned/Unverified |

## Verification Requirements
- Mobile security review checklist.
- Token/session storage inspection.
- API auth negative tests from mobile client perspective.
- Dependency/vulnerability scanning for mobile stack when implemented.

## Related Files
- `../08-mobile/01-android-architecture.md`
- `../08-mobile/03-api-integration.md`
- `../08-mobile/04-offline-sync.md`
