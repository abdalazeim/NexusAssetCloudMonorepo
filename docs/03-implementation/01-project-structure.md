┌──────────────────────────────────────────────────────────────┐
│  File        : 03-implementation/01-project-structure.md    │
│  Project     : NexusAsset Cloud                              │
│  Module      : 03-implementation                             │
│  Version     : 1.0.0                                         │
│  Status      : Stable                                        │
│  Owner       : Platform Architecture                         │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  PROJECT STRUCTURE                                             ║
║  Canonical NexusAsset Cloud Monorepo Layout                    ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Purpose](#1-purpose)
2. [Structure Principles](#2-structure-principles)
3. [Canonical Monorepo Tree](#3-canonical-monorepo-tree)
4. [Backend App Pattern](#4-backend-app-pattern)
5. [Naming and Placement Rules](#5-naming-and-placement-rules)
6. [Dependency Direction](#6-dependency-direction)
7. [Validation Checklist](#7-validation-checklist)
8. [Related Files](#8-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Purpose                                              │
└──────────────────────────────────────────────────────────┘

This document defines the canonical repository structure for NexusAsset Cloud. It normalizes legacy draft names and establishes a stable layout for backend, frontend, mobile, shared packages, API contracts, infrastructure, tests, and enterprise documentation.

┌──────────────────────────────────────────────────────────┐
│  2. Structure Principles                                 │
└──────────────────────────────────────────────────────────┘

- **Monorepo coherence:** Keep API contract, clients, services, and infrastructure aligned.
- **Bounded contexts:** Implement business domains as explicit backend apps (`nxa_*`).
- **Clean Architecture:** Keep `domain`, `application`, `infrastructure`, and `interfaces` separated inside each app.
- **Single source of truth:** Maintain OpenAPI in `api/contracts/openapi.yaml`.
- **Operational realism:** Store deployable infrastructure as code under `infrastructure/`.

┌──────────────────────────────────────────────────────────┐
│  3. Canonical Monorepo Tree                              │
└──────────────────────────────────────────────────────────┘

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ nexusasset-cloud/
│ ├── backend/
│ │   ├── nexusasset/                     # Django project package
│ │   │   ├── settings/                   # base, dev, test, staging, prod
│ │   │   ├── di/                         # Composition root (ports -> adapters)
│ │   │   ├── urls.py
│ │   │   ├── asgi.py
│ │   │   ├── wsgi.py
│ │   │   ├── celery.py
│ │   │   └── logging.py
│ │   └── apps/
│ │       ├── nxa_core/
│ │       ├── nxa_accounts/
│ │       ├── nxa_tenants/
│ │       ├── nxa_assets/
│ │       ├── nxa_lansweeper/
│ │       ├── nxa_billing/
│ │       ├── nxa_notifications/
│ │       └── nxa_audit/
│ ├── frontend/                           # Next.js 15 + React 19 + BFF
│ ├── mobile/android/                     # Kotlin + Jetpack Compose
│ ├── packages/@nexusasset/
│ │   ├── api-client/
│ │   ├── shared-types/
│ │   ├── design-tokens/
│ │   ├── ui/
│ │   ├── eslint-config/
│ │   └── tsconfig/
│ ├── api/contracts/openapi.yaml          # OpenAPI 3.1 contract
│ ├── infrastructure/
│ │   ├── traefik/                        # nxa-traefik-public, nxa-le, TLS, middleware
│ │   ├── docker/                         # nxa-backend, nxa-frontend, nxa-worker, nxa-db, nxa-redis
│ │   ├── ansible/
│ │   ├── jenkins/
│ │   └── monitoring/
│ ├── tests/                              # architecture, contract, e2e, load, security
│ └── 00-overview/..14-appendices/        # Documentation and ADRs
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│  4. Backend App Pattern                                  │
└──────────────────────────────────────────────────────────┘

▸ Required Layout for Every Bounded Context
┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ backend/apps/nxa_<context>/
│ ├── README.md
│ ├── domain/                    # Entities, value objects, events, domain services
│ ├── application/               # Use cases, DTOs, ports
│ ├── infrastructure/            # ORM models, repositories, adapters, mappers
│ ├── interfaces/                # DRF views, serializers, urls, task entrypoints
│ ├── migrations/
│ └── tests/
│     ├── unit/
│     ├── integration/
│     └── api/
└──────────────────────────────────────────────────────────────────────────────┘

▸ Layer Responsibilities
| Layer | Responsibility | Must Not Do |
|---|---|---|
| `domain` | Express business rules and invariants | Import Django/DRF/ORM/infrastructure |
| `application` | Orchestrate use cases through ports | Contain framework-specific persistence logic |
| `infrastructure` | Implement ports/adapters and persistence | Own business rules |
| `interfaces` | Translate HTTP/tasks into application commands/queries | Contain core domain decisions |

┌──────────────────────────────────────────────────────────┐
│  5. Naming and Placement Rules                           │
└──────────────────────────────────────────────────────────┘

| Item | Canonical Name | Placement |
|---|---|---|
| Repository root | `nexusasset-cloud/` | Root |
| Django project package | `nexusasset/` | `backend/nexusasset/` |
| Bounded context apps | `nxa_core`, `nxa_accounts`, `nxa_tenants`, `nxa_assets`, `nxa_lansweeper`, `nxa_billing`, `nxa_notifications`, `nxa_audit` | `backend/apps/` |
| Runtime services | `nxa-backend`, `nxa-frontend`, `nxa-worker`, `nxa-db`, `nxa-redis` | Docker/compose/infrastructure configs |
| Edge naming | `nxa-traefik-public`, `nxa-le` | Traefik/Docker network/resolver config |
| Domains | `app.nexusasset.cloud`, `api.nexusasset.cloud` | DNS/Traefik routing |
| Shared packages | `@nexusasset/*` | `packages/@nexusasset/*` |
| Secret keys/env | `NXA_SECRET_KEY`, `NXA_*` | `.env`/secret store (never committed) |

╭─ TIP ───────────────────────────────────────────────────────╮
│ Add `import-linter` and architecture tests in CI to prevent │
│ new code from violating the canonical structure and the     │
│ Dependency Rule.                                            │
╰─────────────────────────────────────────────────────────────╯

┌──────────────────────────────────────────────────────────┐
│  6. Dependency Direction                                 │
└──────────────────────────────────────────────────────────┘

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ Interfaces  -->  Application  -->  Domain
│                      ^
│                      |
│               Infrastructure (implements ports/adapters)
└──────────────────────────────────────────────────────────────────────────────┘

- `✓` Allowed: `interfaces -> application`, `application -> domain`, `infrastructure -> application/domain ports`.
- `✗` Forbidden: `domain -> django.*`, `domain -> interfaces`, `application -> ORM models`, circular cross-context imports without explicit contracts.

┌──────────────────────────────────────────────────────────┐
│  7. Validation Checklist                                 │
└──────────────────────────────────────────────────────────┘

- `✓` Every bounded context uses the `nxa_` prefix.
- `✓` Every bounded context contains `domain/application/infrastructure/interfaces/tests`.
- `✓` The Django project package is `backend/nexusasset/`.
- `✓` Service names use `nxa-*` in Docker/infrastructure.
- `✓` OpenAPI contract exists at `api/contracts/openapi.yaml`.
- `○` Folder READMEs are created for complex areas.
- `○` Legacy draft files are archived or removed after migration.

┌──────────────────────────────────────────────────────────┐
│  8. Related Files                                        │
└──────────────────────────────────────────────────────────┘
- [02-naming-conventions.md](./02-naming-conventions.md)
- [05-dependency-rule.md](./05-dependency-rule.md)
- [../02-design/04-clean-architecture.md](../02-design/04-clean-architecture.md)
- [../09-api/01-openapi-spec.md](../09-api/01-openapi-spec.md)
- [../05-deployment/02-docker.md](../05-deployment/02-docker.md)

─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: docs@nexusasset.cloud
─────────────────────────────────────────────────────────────
