# NexusAsset Cloud — Workspace Package Metadata Description

**Original artifact:** `package.json` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Describes root JS/TS workspace metadata.

## Inputs / Parameters
Node.js/pnpm and workspace config.

## Outputs / Behavior
Workspace scripts and package-manager declaration.

## Detailed Description
Private workspace `nexusasset-cloud` version 1.0.0 declares pnpm and placeholder scripts for dev/lint/test. Full implementation should align scripts to frontend/packages, Turborepo, OpenAPI codegen, and quality gates.

## NexusAsset Cloud Constraints
- Shared packages use `@nexusasset/*`.
- No secrets in manifests.
