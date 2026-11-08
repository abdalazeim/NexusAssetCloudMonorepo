# NexusAsset Cloud — Frontend Package Manifest Description

**Original artifact:** `package.json` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Defines package metadata and scripts for `@nexusasset/frontend`.

## Inputs / Parameters
Node.js/pnpm workspace tooling and safe frontend environment variables.

## Outputs / Behavior
Runnable workspace scripts and dependency metadata.

## Detailed Description
Private package version 1.0.0 with scripts for Next.js dev, build, start, and lint. Full implementation must include Next.js 15 App Router, React 19, TypeScript, Tailwind CSS, shadcn/ui, testing/typecheck tools, and workspace integration with `@nexusasset/*` packages.

## NexusAsset Cloud Constraints
- Browser-exposed variables must not contain secrets.
- BFF must preserve API contract and authentication boundaries.
