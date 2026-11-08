# NexusAsset Cloud — C4 Context and System Boundaries

## Current State and Evidence
Status: Expanded baseline (Phase B/C). Evidence is Descriptor/Target/Planned unless verified by executable implementation, tests, scans, or runtime output.

## Purpose
Describe NexusAsset Cloud in its environment: users, external systems, and trust boundaries.

## Context (Target)
| Actor/System | Interaction | Notes |
|---|---|---|
| Web users (admin/manager/auditor/operator roles as approved) | Use `app.nexusasset.cloud` via `nxa-frontend`/BFF | Authenticated, tenant-scoped |
| Mobile users | Use approved mobile API flows | Secure token handling required |
| Lansweeper platform | Discovery data via GraphQL/webhooks/bulk/delta as approved | External trust boundary; verify/idempotent |
| Email/SMS/push providers | Notifications (if implemented) | Outbound integration; secrets via `NXA_*` |
| CI/CD/deployment systems | Build/release/deploy | Privileged boundary; approvals/audit |

## Trust Boundaries
Public internet → Traefik edge → BFF/API → data stores/workers → external integrations. All boundaries require authentication/authorization, validation, secret protection, and observability as applicable.

## Status
Diagram/detail is target architecture; verify against implemented topology.
