# NexusAsset Cloud — Developer Command Interface Description

**Original artifact:** `Makefile` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Describes canonical `nxa-*` developer/automation targets.

## Inputs / Parameters
Developer shell, toolchain, Docker/Compose, repository paths.

## Outputs / Behavior
Standardized command entrypoints and exit status.

## Detailed Description
`nxa-help` prints targets; `nxa-dev` starts Compose with build; `nxa-test-backend`, `nxa-test-frontend`, `nxa-lint`, and `nxa-codegen` are baseline targets to be fully implemented for pytest/Vitest/Playwright, lint/type/architecture gates, and OpenAPI code generation. Add `nxa-typecheck`, `nxa-openapi-validate`, `nxa-arch-check`, `nxa-down`, and `nxa-logs` as needed.

## NexusAsset Cloud Constraints
- Align commands with docs and CI.
- Do not print or persist secrets.
