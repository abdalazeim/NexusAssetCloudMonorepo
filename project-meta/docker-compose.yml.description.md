# NexusAsset Cloud — Local Runtime Composition Description

**Original artifact:** `docker-compose.yml` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Describes Docker Compose services for NexusAsset Cloud local/skeleton runtime.

## Inputs / Parameters
Docker/Compose, `NXA_*` env, build contexts, host/TLS config.

## Outputs / Behavior
Local orchestrated NexusAsset Cloud topology.

## Detailed Description
Defines `nxa-backend`, `nxa-frontend`, `nxa-worker` (dev images), `nxa-db` (PostgreSQL 16 with local DB/user/password environment values; password must become secret-managed), and `nxa-redis` (Redis 7), grouped under a skeleton profile. Full implementation adds build contexts, networks, Traefik labels, health checks, volumes, dependencies, migrations/startup ordering, and profiles.

## NexusAsset Cloud Constraints
- Use canonical `nxa-*` service names.
- Replace plaintext DB password with secret-managed configuration.
- Route `app.nexusasset.cloud` and `api.nexusasset.cloud` via Traefik/TLS-ready config.
