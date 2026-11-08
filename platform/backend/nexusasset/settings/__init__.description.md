# NexusAsset Cloud — Backend Settings Package Marker

**Original artifact:** `__init__.description.md` (converted to English description; executable/config source removed per project policy).

## Purpose
Marks the backend settings package for NexusAsset Cloud.

## Detailed Description
This package groups environment-specific Django settings modules (for example base, development, test, staging, production) and shared settings initialization. Configuration values must come from `NXA_*` environment variables or approved secret stores.

## NexusAsset Cloud Constraints
- No secrets committed.
- Use `NXA_*` naming.
- Support PostgreSQL, Redis, Celery, JWT, CORS/CSRF, logging, and tenant-safe defaults as implemented.
