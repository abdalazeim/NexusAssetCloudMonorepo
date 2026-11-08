┌──────────────────────────────────────────────────────────────┐
│  File        : CONVENTIONS.md                                │
│  Project     : NexusAsset Cloud                              │
│  Module      : Governance                                    │
│  Version     : 1.0.0                                         │
│  Status      : Stable                                        │
│  Owner       : Platform Engineering                          │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  CONVENTIONS                                                   ║
║  NexusAsset Cloud Engineering and Documentation Standards      ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Purpose](#1-purpose)
2. [Canonical Naming](#2-canonical-naming)
3. [Architecture Conventions](#3-architecture-conventions)
4. [API and Data Conventions](#4-api-and-data-conventions)
5. [Git and Release Conventions](#5-git-and-release-conventions)
6. [Documentation Conventions](#6-documentation-conventions)
7. [Related Files](#7-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Purpose                                              │
└──────────────────────────────────────────────────────────┘
This document defines mandatory conventions for NexusAsset Cloud code, infrastructure, APIs, and documentation.

┌──────────────────────────────────────────────────────────┐
│  2. Canonical Naming                                     │
└──────────────────────────────────────────────────────────┘
| Category | Convention |
|---|---|
| Repository | `nexusasset-cloud/` |
| Django package | `backend/nexusasset/` |
| Bounded contexts | `nxa_core`, `nxa_accounts`, `nxa_tenants`, `nxa_assets`, `nxa_lansweeper`, `nxa_billing`, `nxa_notifications`, `nxa_audit` |
| Runtime services | `nxa-backend`, `nxa-frontend`, `nxa-worker`, `nxa-db`, `nxa-redis` |
| Edge | `nxa-traefik-public`, `nxa-le` |
| Domains | `app.nexusasset.cloud`, `api.nexusasset.cloud` |
| Packages | `@nexusasset/*` |
| Secrets | `NXA_*` (never committed) |

┌──────────────────────────────────────────────────────────┐
│  3. Architecture Conventions                             │
└──────────────────────────────────────────────────────────┘
- Use Clean Architecture: `interfaces -> application -> domain`; infrastructure implements ports.
- Do not import Django/DRF/ORM into Domain.
- Use bounded contexts and explicit integration contracts.
- Enforce rules with `import-linter` and architecture tests.

┌──────────────────────────────────────────────────────────┐
│  4. API and Data Conventions                             │
└──────────────────────────────────────────────────────────┘
- Treat `api/contracts/openapi.yaml` as the single source of truth.
- Use OpenAPI 3.1, JWT, pagination/filtering allowlists, and a unified error envelope.
- Enforce multi-tenancy with `tenant_id` and PostgreSQL RLS.
- Design sync/billing/webhook operations to be idempotent.

┌──────────────────────────────────────────────────────────┐
│  5. Git and Release Conventions                          │
└──────────────────────────────────────────────────────────┘
- Branches: `feat/`, `fix/`, `chore/`, `docs/`, `refactor/`, `hotfix/`.
- Commits: Conventional Commits.
- Releases: Semantic Versioning (SemVer) and maintained `CHANGELOG.md`.

┌──────────────────────────────────────────────────────────┐
│  6. Documentation Conventions                            │
└──────────────────────────────────────────────────────────┘
- Write documentation in English (except explicit Arabic executive summaries).
- Include header, Table of Contents (ToC), content, related files, and footer.
- Use NexusAsset Cloud naming and approved terminology.

┌──────────────────────────────────────────────────────────┐
│  7. Related Files                                        │
└──────────────────────────────────────────────────────────┘
- [03-implementation/02-naming-conventions.md](./03-implementation/02-naming-conventions.md)
- [03-implementation/05-dependency-rule.md](./03-implementation/05-dependency-rule.md)
- [11-governance/01-git-workflow.md](./11-governance/01-git-workflow.md)

─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: docs@nexusasset.cloud
─────────────────────────────────────────────────────────────
