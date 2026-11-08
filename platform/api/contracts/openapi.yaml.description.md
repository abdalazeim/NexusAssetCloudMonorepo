# NexusAsset Cloud — OpenAPI 3.1 Contract Description

**Original artifact:** `openapi.yaml` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Defines the NexusAsset Cloud REST API contract, the source of truth for backend, BFF, frontend/mobile clients, and code generation.

## Inputs / Parameters
HTTP requests matching operations, parameters, and security schemes.

## Outputs / Behavior
HTTP responses matching schemas, status codes, and error envelopes.

## Detailed Description
The current contract is OpenAPI 3.1.0 for "NexusAsset Cloud API" version 1.0.0, server `https://api.nexusasset.cloud`, with `GET /healthz` returning HTTP 200. The full contract must include JWT security schemes, common error schemas, pagination/filter/sort conventions, tenant-aware access rules, authentication/bootstrap endpoints, assets/CMDB endpoints, Lansweeper integration and sync-status endpoints, audit/notification endpoints, mobile bootstrap endpoints, and stable operation IDs.

## NexusAsset Cloud Constraints
- OpenAPI 3.1 is authoritative.
- Breaking changes require explicit versioning/deprecation.
- Tenant scope must never be accepted from untrusted client input for authorization.
