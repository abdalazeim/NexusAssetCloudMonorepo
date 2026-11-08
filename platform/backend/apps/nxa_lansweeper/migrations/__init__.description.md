# NexusAsset Cloud — Backend Package Marker — nxa_lansweeper/migrations

**Original artifact:** `__init__.description.md` (converted to English description; executable/config source removed per project policy).

## Purpose
Marks the `migrations` package for the `nxa_lansweeper` bounded context as a Python package boundary in the NexusAsset Cloud backend.

## Detailed Description
This package belongs to `nxa_lansweeper` and represents the `migrations` layer in Clean Architecture. It should expose only symbols appropriate to that layer and preserve the dependency rule: interfaces may depend on application, application on domain, and infrastructure may implement application ports; domain must not depend on Django, DRF, ORM, or delivery frameworks.

## NexusAsset Cloud Constraints
- Canonical bounded context naming uses `nxa_*`.
- Keep layer responsibilities explicit and testable.
- Tenant-aware behavior (`tenant_id` and RLS alignment) must be enforced in appropriate layers, not bypassed.
