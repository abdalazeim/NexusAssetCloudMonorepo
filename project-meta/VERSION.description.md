# NexusAsset Cloud — Version Identifier Description

**Original artifact:** `VERSION` (converted to English description; original file removed per approved scope 1).

## Purpose
Stores the canonical NexusAsset Cloud release version identifier.

## Inputs / Parameters
Release/tooling scripts and maintainers.

## Outputs / Behavior
A single version string for release coordination.

## Detailed Description
The value represents the current project version (baseline 1.0.0 unless formally changed). Release automation, changelog, image tags, and docs should reference it per the release process.

## NexusAsset Cloud Constraints
- Update only through the approved release/versioning process.
- Keep synchronized with changelog and package/API versions where applicable.
