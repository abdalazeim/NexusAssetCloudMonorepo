# NexusAsset Cloud — Django Project Package Marker

**Original artifact:** `__init__.description.md` (converted to English description; executable/config source removed per project policy).

## Purpose
Marks `nexusasset` as the canonical Django project package for NexusAsset Cloud.

## Detailed Description
The package contains project-level configuration: settings modules, URL routing, ASGI/WSGI entry modules, Celery application wiring, logging/composition utilities, and dependency-injection/composition-root modules as needed.

## NexusAsset Cloud Constraints
- Canonical backend package path is `backend/nexusasset/`.
- Runtime service names use `nxa-*` in deployment descriptors.
