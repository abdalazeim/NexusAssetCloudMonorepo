# NexusAsset Cloud — Backend Python Project Metadata Description

**Original artifact:** `pyproject.toml` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Defines Python packaging and tool metadata for `nxa-backend`.

## Inputs / Parameters
Python build/install tooling and environment.

## Outputs / Behavior
Package metadata, dependency inputs, and tool configuration.

## Detailed Description
Project `nxa-backend` version 1.0.0 requires Python 3.12+ and declares dependencies on Django 5.x, Django REST Framework, PostgreSQL driver (`psycopg` binary extra), Redis client, and Celery. Pytest is configured for `nexusasset.settings.test`. Full metadata should add dev dependencies/config for ruff, black, mypy, import-linter, coverage, and security/contract tooling.

## NexusAsset Cloud Constraints
- Support Clean Architecture, PostgreSQL/RLS, Celery/Redis, and secure API delivery.
- Vulnerability scanning and lock/pin strategy are handled by quality/security phases.
