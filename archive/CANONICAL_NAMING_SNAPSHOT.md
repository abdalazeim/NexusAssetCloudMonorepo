# NexusAsset Cloud — Canonical Naming Snapshot

## Purpose
Preserved snapshot of naming conventions that must remain stable across NexusAsset Cloud documentation and future implementation.

## Naming Matrix
| Area | Canonical Value / Pattern |
|---|---|
| Product | NexusAsset Cloud |
| Tagline | "Where Discovery Meets Enterprise" |
| Django project package | `backend/nexusasset/` (target) |
| Bounded contexts | `nxa_core`, `nxa_accounts`, `nxa_tenants`, `nxa_assets`, `nxa_lansweeper`, `nxa_billing`, `nxa_notifications`, `nxa_audit` |
| Runtime services | `nxa-backend`, `nxa-frontend`, `nxa-worker`, `nxa-db`, `nxa-redis` |
| Edge / TLS | `nxa-traefik-public`, `nxa-le` |
| Domains | `app.nexusasset.cloud`, `api.nexusasset.cloud` |
| Shared packages | `@nexusasset/*` |
| Secrets / environment variables | `NXA_*` |
| API contract (target) | OpenAPI 3.1 source of truth; current descriptor at `platform/api/contracts/openapi.yaml.description.md` |

## Architectural Naming Rules
- Bounded contexts use `nxa_` (Python/module boundary).
- Runtime/container/service identities use `nxa-`.
- Shared JS/TS packages use `@nexusasset/`.
- Secrets/config keys use `NXA_` and are never committed.
- Tenant-owned data uses `tenant_id` and PostgreSQL RLS-aligned controls.
