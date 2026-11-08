# NexusAsset Cloud — Native Mobile Architecture and Delivery

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Baseline documentation expanded (Phase B/C in progress) |
| Evidence level | Target/Descriptor Evidence; Android/iOS implementation not verified as runnable in current repository state |
| Primary evidence | `prompts/06-mobile-android.md`, `platform/mobile/android/*` descriptors, `docs/08-mobile/*`, `docs/09-api/*` |
| Constraint | Do not claim offline sync, push delivery, or store release readiness unless implementation evidence exists. |

## Purpose
Define NexusAsset Cloud mobile architecture and delivery standards for API-consuming clients, with Android as the primary baseline and iOS as a target/planned track where specified.

## Mobile Principles
1. Mobile clients consume approved NexusAsset Cloud API contracts; no hidden/private contract assumptions.
2. Tenant and authorization boundaries are enforced server-side; mobile stores only necessary authorized context.
3. Sensitive tokens/credentials are handled using secure platform storage mechanisms (target).
4. Network failures, partial sync, and empty/error states are explicit user-visible states.
5. Offline behavior is documented as Implemented only when local storage/conflict resolution/retry logic is verified.

## Android Baseline (Target)
| Area | Requirement | Status |
|---|---|---|
| Architecture | Kotlin, clear separation (UI/domain/data or Clean/MVVM as approved), testable modules | Target/Unverified |
| API integration | OpenAPI-aligned models/clients, auth bootstrap, error handling | Target/Unverified |
| Auth/token | Secure token storage, refresh/expiry handling, logout behavior | Target/Unverified |
| Asset flows | List/search/detail and relevant operational views per MVP | Target/Unverified |
| Offline sync | Local cache/outbox/conflict resolution only if implemented | Planned/Unverified |
| Push notifications | Registration, permission handling, deep links only if implemented | Planned/Unverified |
| Release | Build variants, signing, versioning, staged rollout, force-update policy if used | Target/Unverified |

## Contents
| File | Description | Audit note |
|---|---|---|
| `01-android-architecture.md` | Android architecture | Expand with modules/layers, navigation, state, dependencies. |
| `02-ios-architecture.md` | iOS architecture | Keep clearly labeled Target/Planned unless implemented. |
| `03-api-integration.md` | Mobile API integration/bootstrap | Expand with auth, pagination, retries, error mapping. |
| `04-offline-sync.md` | Offline cache/outbox/conflict resolution | Mark Unverified/Planned unless evidence exists. |
| `05-push-notifications.md` | Push notifications | Mark Unverified/Planned unless evidence exists. |
| `06-release-process.md` | Mobile release/force update | Expand with build/signing/store pipeline requirements. |

## Related Files
- `../09-api/README.md`
- `../07-security/06-mobile-security.md`
- `../../prompts/06-mobile-android.md`
