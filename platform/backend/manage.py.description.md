# NexusAsset Cloud — Django Management Entrypoint Description

**Original artifact:** `manage.py.description.md` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Provides the administrative command entrypoint for the NexusAsset Cloud Django backend (`nxa-backend`).

## Inputs / Parameters
Command-line arguments and environment variables, especially `DJANGO_SETTINGS_MODULE` and `NXA_*` runtime configuration values.

## Outputs / Behavior
Executes the requested Django management command in the configured environment.

## Detailed Description
The implementation sets the default Django settings module to the NexusAsset Cloud development settings module (under `backend/nexusasset/settings/`) when not already provided, imports Django's command-line execution helper, and forwards command-line arguments. It is used for migrations, checks, shell, custom management commands (including Lansweeper sync commands), and other administrative tasks.

## NexusAsset Cloud Constraints
- Default settings path must resolve to canonical package `nexusasset.settings.*`.
- Administrative commands must respect tenant safety and must not leak secrets.
- Use only in approved environments; production operations follow runbooks and CI/CD controls.
