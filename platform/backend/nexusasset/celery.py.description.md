# NexusAsset Cloud — Celery Application Description

**Original artifact:** `celery.py.description.md` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Defines the Celery application used by the NexusAsset Cloud background worker service (`nxa-worker`).

## Inputs / Parameters
`NXA_*` environment variables, Celery broker/result backend settings (typically Redis), Django settings, and registered task modules.

## Outputs / Behavior
A configured Celery application instance used by worker and beat processes.

## Detailed Description
The implementation creates/configures a Celery app for the NexusAsset Cloud Django project, loads Django settings, discovers asynchronous tasks from installed `nxa_*` bounded-context apps, and exposes the app for worker/beat processes. It supports scheduled synchronization (including Lansweeper sync jobs), notifications, billing/background processing, and other idempotent asynchronous workloads.

## NexusAsset Cloud Constraints
- Tasks touching tenant data must enforce tenant scope.
- Lansweeper and external integration tasks must be idempotent, retry-safe, rate-limit aware, and auditable.
- Secrets are supplied through approved secret stores/environment variables only.
