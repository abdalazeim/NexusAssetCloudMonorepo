# NexusAsset Cloud — Backend Container Image Description

**Original artifact:** `Dockerfile` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Describes the container build for `nxa-backend`.

## Inputs / Parameters
Build context, backend source, dependency metadata, runtime `NXA_*` env vars.

## Outputs / Behavior
Runnable backend container image on port 8000.

## Detailed Description
Based on slim Python 3.12, working directory `/app`, Python runtime flags for clean/unbuffered execution, copies metadata, installs project dependencies, copies source, exposes port 8000, and starts Gunicorn serving `nexusasset.wsgi:application` on `0.0.0.0:8000`. Production hardening adds non-root user, pinned deps, minimal layers, and health checks.

## NexusAsset Cloud Constraints
- Inject secrets at runtime.
- Scan and harden before release.
