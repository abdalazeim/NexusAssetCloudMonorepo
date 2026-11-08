# NexusAsset Cloud — Backend Package Marker — nxa_core/domain

**Original artifact:** `__init__.description.md` (converted to English description; executable/config source removed per project policy).

## Purpose
Marks the `domain` package for the `nxa_core` bounded context as a Python package boundary in the NexusAsset Cloud backend.

## Detailed Description
This package belongs to `nxa_core` and represents the `domain` layer in Clean Architecture. It should expose only symbols appropriate to that layer and preserve the dependency rule: interfaces may depend on application, application on domain, and infrastructure may implement application ports; domain must not depend on Django, DRF, ORM, or delivery frameworks.

## NexusAsset Cloud Constraints
- Canonical bounded context naming uses `nxa_*`.
- Keep layer responsibilities explicit and testable.
- Tenant-aware behavior (`tenant_id` and RLS alignment) must be enforced in appropriate layers, not bypassed.
