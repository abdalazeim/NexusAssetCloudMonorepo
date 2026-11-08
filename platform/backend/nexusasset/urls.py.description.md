# NexusAsset Cloud — Backend URL Routing Description

**Original artifact:** `urls.py.description.md` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Defines top-level HTTP routing for the NexusAsset Cloud backend API (`nxa-backend`).

## Inputs / Parameters
Incoming HTTP requests.

## Outputs / Behavior
HTTP responses, including health status JSON and routed API responses.

## Detailed Description
The current baseline exposes a health-check endpoint at `/healthz/` that returns a JSON response indicating service health/status for the backend. Future routing must include versioned/API routes, authentication endpoints, bounded-context interfaces, webhook endpoints (including Lansweeper webhooks with verification), and observability/admin routes as approved, all aligned to `api/contracts/openapi.yaml`.

## NexusAsset Cloud Constraints
- Health endpoints support container/orchestrator probes.
- Public and webhook routes must enforce security controls (authentication, signature verification, rate limits) as applicable.
- Routes and schemas must match OpenAPI 3.1 source of truth.
