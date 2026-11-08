# NexusAsset Cloud — C4 Containers and Runtime Topology

## Current State and Evidence
Status: Expanded baseline (Phase B/C). Evidence is Descriptor/Target/Planned unless verified by executable implementation, tests, scans, or runtime output.

## Containers (Target)
| Container/Service | Responsibility | Key dependencies | Status |
|---|---|---|---|
| `nxa-frontend` | Next.js UI + BFF boundary | API, auth/session config | Descriptor/Target |
| `nxa-backend` | Django/DRF API, use cases, domain orchestration | PostgreSQL, Redis, integrations | Descriptor/Target |
| `nxa-worker` | Celery async jobs (sync, notifications, scheduled tasks) | Redis broker, DB, external APIs | Descriptor/Target |
| `nxa-db` | PostgreSQL 16 data, RLS target | Storage/backup | Descriptor/Target |
| `nxa-redis` | Cache/broker | Memory/persistence config | Descriptor/Target |
| `nxa-traefik-public` / `nxa-le` | Edge routing/TLS | Certs, DNS, service network | Descriptor/Target |

## Runtime Concerns
Health/readiness, migrations, worker scaling, secret injection, tenant-safe configuration, logging/metrics/traces, backup/restore.

## Status
Target topology; runnable proof Unverified in current state.
