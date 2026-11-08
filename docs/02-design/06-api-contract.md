# NexusAsset Cloud — OpenAPI Contract and API Standards

## Current State and Evidence
Status: Expanded baseline (Phase B/C). Evidence is Descriptor/Target/Planned unless verified by executable implementation, tests, scans, or runtime output.

## Status
Target/Descriptor Evidence. Current descriptor: `platform/api/contracts/openapi.yaml.description.md` (skeleton; target contract path `api/contracts/openapi` + YAML).

## API Standards (Target)
- OpenAPI 3.1 source of truth.
- Consistent resource naming, operation IDs, tags, and bounded-context alignment.
- Explicit auth requirements and tenant scoping rules.
- Standard pagination/filter/sort and error envelope.
- Idempotency for create/sync/trigger operations prone to retries.
- Backward-compatible evolution where possible; deprecation/versioning for breaking changes.

## BFF vs Public/Internal API
BFF endpoints may tailor responses for web UX but must not bypass backend authorization, tenant isolation, or contract discipline. Public/mobile contracts must be documented and governed.

## Evidence Needed
Complete OpenAPI spec, implementation parity, contract tests, and codegen outputs.
