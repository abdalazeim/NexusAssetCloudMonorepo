# Phase 06 — Mobile (Android Kotlin)

## Objective
Implement the NexusAsset Cloud Android baseline in `mobile/android/` using Kotlin and modern Android architecture, consuming NexusAsset Cloud APIs.

## Preconditions
- Phases 01–03 complete; mobile-relevant OpenAPI paths and auth/bootstrap behavior are defined.

## Required Context
- `mobile/android/`
- `api/contracts/openapi.yaml`
- `08-mobile/` docs
- Auth/tenant/bootstrap API contracts

## Deliverables
1. **Project baseline**
   - Kotlin + Gradle structure, modules/package naming aligned to NexusAsset Cloud, build variants, and environment/config handling.
   - Network/client layer aligned to OpenAPI contracts (generated models/client or typed equivalent).
2. **Architecture**
   - Use a clear mobile architecture (e.g., Clean-ish layers/ViewModel/repository) with separation of UI, domain/use-case logic, and data/network adapters.
   - Secure token/session storage patterns; no secrets committed.
3. **Core flows (baseline)**
   - App bootstrap/configuration.
   - Authentication and tenant/context selection if applicable.
   - Asset lookup/list/detail baseline.
   - Sync/status or field operations relevant to Lansweeper/CMDB mobile use cases where API supports them.
   - Offline/error/loading/empty handling for critical screens.
4. **Quality**
   - Unit tests for repositories/use cases/mappers; basic UI/instrumentation test scaffolding as feasible.
   - Lint/static checks configuration.

## Verification
┌─ Specification Block (converted from `powershell`; scope 2 — no fenced code blocks) ─┐
│ cd mobile/android
│ ./gradlew tasks
│ ./gradlew test  # or assemble/test variants available in environment
│ ./gradlew lint  # if configured
└──────────────────────────────────────────────────────────────────────────────┘
If Android SDK/Gradle tooling is unavailable, ensure project files are complete, document exact commands, and run static consistency checks.

## Exit Criteria
- Android project is structurally complete and buildable in a properly configured Android environment.
- API/auth/bootstrap flows are wired to contract patterns.
- Core asset/mobile baseline screens and tests/scaffolding exist.
