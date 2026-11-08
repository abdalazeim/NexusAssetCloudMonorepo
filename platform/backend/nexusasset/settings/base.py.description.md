# NexusAsset Cloud — Django Base Settings Description

**Original artifact:** `base.py.description.md` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Provides shared Django base settings for NexusAsset Cloud (`nxa-backend`).

## Inputs / Parameters
Environment variables and secret references (`NXA_SECRET_KEY`, `NXA_DATABASE_URL`, `NXA_REDIS_URL`, allowed hosts, debug flags, JWT settings, Lansweeper settings, observability settings).

## Outputs / Behavior
A Django settings module used as the foundation for development, test, staging, and production settings.

## Detailed Description
The base settings establish project paths, a development-unsafe placeholder secret key marker, debug disabled by default, empty allowed hosts, minimal installed applications including Django contenttypes/auth and Django REST Framework, root URL configuration, and an empty database mapping in the skeleton. The full implementation must load `NXA_*` configuration; configure PostgreSQL 16 and Redis; enable DRF/JWT/authentication; configure CORS/CSRF/security headers; register all `nxa_*` apps; configure Celery, logging, OpenTelemetry hooks, static/media handling, and tenant-aware/RLS-compatible database behavior as required.

## NexusAsset Cloud Constraints
- Never hardcode real secrets; `NXA_SECRET_KEY` and similar values must come from secure configuration.
- Production must set explicit allowed hosts and secure transport/cookie settings.
- Tenant isolation and audit/security requirements apply to all data access paths.
