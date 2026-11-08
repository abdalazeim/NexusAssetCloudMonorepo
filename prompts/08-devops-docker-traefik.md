# Phase 08 — DevOps (Docker + Traefik)

## Objective
Implement a working local runtime for NexusAsset Cloud using Docker Compose and Traefik, with canonical service names, networking, health checks, and TLS-ready routing.

## Preconditions
- Phases 01–07 complete enough for services to build/start.
- Phase 09/10 hardening must not be treated as production-ready until local runtime is verified.

## Required Context
- `docker-compose.yml`
- `backend/Dockerfile`, `frontend/Dockerfile`
- `infrastructure/docker/`, `infrastructure/traefik/`
- `.env.example`
- `05-deployment/` docs

## Deliverables
1. **Compose/runtime services**
   - Define/align services: `nxa-backend`, `nxa-frontend`, `nxa-worker`, `nxa-db`, `nxa-redis`, and Traefik edge services (`nxa-traefik-public`/related naming as adopted) with clear dev/prod profiles.
   - Configure build contexts, env files, volumes, networks, restart policies, health checks, and dependency ordering.
2. **Traefik routing**
   - Route `app.nexusasset.cloud` to frontend and `api.nexusasset.cloud` to backend (local override via hosts/env as documented).
   - Configure HTTP->HTTPS redirection, TLS resolver (`nxa-le` naming), secure headers middleware, and reasonable rate limits.
3. **Runtime configuration**
   - PostgreSQL initialization, migrations/startup command strategy, Celery worker/beat startup, static/media handling as needed.
   - Use `NXA_*` env vars; provide safe defaults in `.env.example` only.
4. **Operational scripts**
   - Add/align `nxa-dev`, `nxa-down`, `nxa-logs`, `nxa-migrate`, `nxa-seed` (if applicable), and health-check commands.

## Verification
┌─ Specification Block (converted from `powershell`; scope 2 — no fenced code blocks) ─┐
│ copy .env.example .env  # if needed; do not commit .env
│ docker compose config
│ docker compose up --build -d
│ docker compose ps
│ # curl/Invoke-WebRequest health endpoints and routed hosts as configured
│ docker compose logs --tail=100 nxa-backend nxa-frontend nxa-worker
└──────────────────────────────────────────────────────────────────────────────┘

## Exit Criteria
- `docker compose config` succeeds.
- Core `nxa-*` services start and report healthy (or failures are diagnosed/fixed).
- Traefik routes frontend/API locally with documented host/TLS behavior.
- No secrets are hardcoded.
