# NexusAsset Cloud — Turborepo Pipeline Description

**Original artifact:** `turbo.json` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Defines Turborepo task pipeline behavior.

## Inputs / Parameters
Turborepo and workspace scripts.

## Outputs / Behavior
Task ordering, caching, and outputs.

## Detailed Description
Build depends on upstream builds and caches `dist/**` and `.next/**`; lint and test tasks are declared. Add typecheck/codegen/format tasks and safe caching rules as needed.

## NexusAsset Cloud Constraints
- Do not cache secrets or sensitive environment-specific data.
