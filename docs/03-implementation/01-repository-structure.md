# ═══════════════════════════════════════════════════════════════
#  NEXUSASSET CLOUD — COMPLETE REPOSITORY STRUCTURE
# ═══════════════════════════════════════════════════════════════
#  Document ID        : RS-001
#  Version            : 1.0.0
#  Status             : Target Architecture (Design Baseline)
#  Classification     : Internal — Architecture & Engineering
#  Owner              : Platform Architecture
#  Last Reviewed      : 2026-11-09
#  Next Review        : 2026-12-09
#  Source Evidence    : Platform Architecture Decisions, ADR 0001–0010
#  Canonical Repo     : nexusasset-cloud/
#  Domains            : app.nexusasset.cloud  |  api.nexusasset.cloud
#  Package Scope      : @nexusasset/*
#  Secret Prefix      : NXA_*
# ═══════════════════════════════════════════════════════════════

# NexusAsset Cloud — Complete Repository Structure
**Target Monorepo Layout for Enterprise CMDB + Lansweeper Integration**

> **Purpose** — Define the complete, canonical repository structure for NexusAsset Cloud. This is the **target architecture** — current repository state may differ. All paths use forward slashes; directory names are case-sensitive. This document supersedes legacy drafts and establishes the single source of truth for repository layout.

---

## Table of Contents
1. [Structure Principles](#1-structure-principles)
2. [Root Level](#2-root-level)
3. [Documentation (`docs/`)](#3-documentation-docs)
4. [Execution Prompts (`prompts/`)](#4-execution-prompts-prompts)
5. [Platform (`platform/`) — Core Implementation](#5-platform-platform--core-implementation)
   - [5.1 Backend (`platform/backend/`)](#51-backend-platformbackend)
   - [5.2 Frontend (`platform/frontend/`)](#52-frontend-platformfrontend)
   - [5.3 Mobile (`platform/mobile/`)](#53-mobile-platformmobile)
   - [5.4 Shared Packages (`platform/packages/@nexusasset/`)](#54-shared-packages-platformpackagesnexusasset)
   - [5.5 Contracts (`platform/contracts/`)](#55-contracts-platformcontracts)
   - [5.6 Design System (`platform/design-system/`)](#56-design-system-platformdesign-system)
   - [5.7 Infrastructure (`platform/infrastructure/`)](#57-infrastructure-platforminfrastructure)
   - [5.8 Deployment (`platform/deployment/`)](#58-deployment-platformdeployment)
   - [5.9 Security (`platform/security/`)](#59-security-platformsecurity)
   - [5.10 Observability (`platform/observability/`)](#510-observability-platformobservability)
   - [5.11 Operations (`platform/ops/`)](#511-operations-platformops)
   - [5.12 Tests (`platform/tests/`)](#512-tests-platformtests)
   - [5.13 Quality (`platform/quality/`)](#513-quality-platformquality)
   - [5.14 Scripts (`platform/scripts/`)](#514-scripts-platformscripts)
6. [Project Metadata (`project-meta/`)](#6-project-metadata-project-meta)
7. [Archive (`archive/`)](#7-archive-archive)
8. [Naming & Placement Rules](#8-naming--placement-rules)
9. [Validation Checklist](#9-validation-checklist)

---

## 1. Structure Principles

┌─────────────────────────────────────────────────────────────────────┐
│ ▸ MONOREPO COHERENCE                                                 │
│   Keep API contract, clients, services, and infrastructure aligned  │
│   in a single repository with clear boundaries.                     │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ BOUNDED CONTEXTS                                                   │
│   Implement business domains as explicit backend apps (`nxa_*`)     │
│   following Clean Architecture layering.                            │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ CLEAN ARCHITECTURE                                                 │
│   Keep `domain`, `application`, `infrastructure`, `interfaces`      │
│   separated inside each bounded context. Dependency direction:      │
│   `Interfaces → Application → Domain` (inward only).                │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ SINGLE SOURCE OF TRUTH                                             │
│   OpenAPI 3.1 contract at `platform/contracts/openapi/openapi.yaml` │
│   is the authoritative API specification.                           │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ OPERATIONAL REALISM                                                │
│   Store deployable infrastructure as code under                     │
│   `platform/infrastructure/` and `platform/deployment/`.            │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ DOCUMENTATION AS CODE                                              │
│   All architecture, design, ADRs, runbooks, and guides in `docs/`   │
│   following NexusAsset Cloud documentation standards.               │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ EXECUTION PROMPTS                                                  │
│   Phased implementation prompts in `prompts/` (00–20) for          │
│   reproducible, session-based development.                          │
└─────────────────────────────────────────────────────────────────────┘

---

## 2. Root Level

```text
nexusasset-cloud/
├── README.md                           # Project overview, quick links
├── CHANGELOG.md                        # Version history (Keep a Changelog)
├── CONTRIBUTING.md                     # Contribution guidelines
├── CODE_OF_CONDUCT.md                  # Community standards
├── SECURITY.md                         # Security reporting, policies
├── SUPPORT.md                          # Support channels, SLAs
├── LICENSE                             # License text
├── Makefile                            # Common development commands
│
├── .editorconfig                       # Editor consistency
├── .gitattributes                      # Git attributes (LFS, diff, merge)
├── .gitignore                          # Git ignore rules
├── .dockerignore                       # Docker build context exclusions
├── .pre-commit-config.yaml             # Pre-commit hooks
├── .env.example                        # Environment variable template
├── .npmrc                              # npm/pnpm configuration
├── .python-version                     # Python version (pyenv/asdf)
├── .node-version                       # Node version (nvm/fnm)
│
├── pyproject.toml                      # Python project config (uv/poetry)
├── uv.lock                             # Python lockfile
├── package.json                        # Root package.json (workspaces)
├── pnpm-lock.yaml                      # pnpm lockfile
├── pnpm-workspace.yaml                 # pnpm workspace config
├── turbo.json                          # Turborepo pipeline config
│
├── compose.yaml                        # Docker Compose base
├── compose.dev.yaml                    # Development overrides
├── compose.staging.yaml                # Staging overrides
├── compose.prod.yaml                   # Production overrides
│
├── project-meta/                       # Governance, policies, runtime descriptors
├── docs/                               # Enterprise documentation (00–22)
├── prompts/                            # Phased execution prompts (00–20)
├── platform/                           # Core implementation (backend, frontend, mobile, infra, etc.)
└── archive/                            # Historical inputs, traceability, snapshots
```

---

## 3. Documentation (`docs/`)

```text
docs/
├── 00-overview/                        # Vision, glossary, quick-start
│   ├── README.md
│   ├── 01-vision.md
│   ├── 02-glossary.md
│   └── 03-quick-start.md
│
├── 01-business-analysis/               # Business context, stakeholders, value
│   ├── README.md
│   ├── 01-stakeholders.md
│   ├── 02-value-proposition.md
│   └── 03-success-criteria.md
│
├── 02-requirements/                    # Functional & non-functional requirements
│   ├── README.md
│   ├── 01-functional-requirements.md
│   ├── 02-non-functional-requirements.md
│   └── 03-acceptance-criteria.md
│
├── 03-system-architecture/             # High-level architecture (C4 L1–L2)
│   ├── README.md
│   ├── 01-c4-context.md
│   ├── 02-c4-container.md
│   └── 03-deployment-topology.md
│
├── 04-domain-design/                   # Domain modeling, bounded contexts
│   ├── README.md
│   ├── 01-bounded-contexts.md
│   ├── 02-domain-events.md
│   ├── 03-aggregates.md
│   └── 04-context-map.md
│
├── 05-implementation/                  # Implementation standards, structure
│   ├── README.md
│   ├── 01-repository-structure.md      # ← THIS FILE
│   ├── 02-naming-conventions.md
│   ├── 03-coding-standards.md
│   ├── 04-clean-architecture.md
│   └── 05-dependency-rule.md
│
├── 06-testing-quality/                 # Testing strategy, quality gates
│   ├── README.md
│   ├── 01-testing-strategy.md
│   ├── 02-test-pyramid.md
│   ├── 03-contract-testing.md
│   ├── 04-architecture-tests.md
│   └── 05-quality-gates.md
│
├── 07-deployment/                      # Deployment architecture, Docker, K8s
│   ├── README.md
│   ├── 01-docker.md
│   ├── 02-kubernetes.md
│   ├── 03-environments.md
│   └── 04-rollout-strategies.md
│
├── 08-operations/                      # Runbooks, incident management, ops
│   ├── README.md
│   ├── 01-runbooks.md
│   ├── 02-incident-management.md
│   ├── 03-backup-restore.md
│   ├── 04-capacity-planning.md
│   └── 05-maintenance.md
│
├── 09-security/                        # Security architecture, threat model
│   ├── README.md
│   ├── 01-threat-model.md
│   ├── 02-security-controls.md
│   ├── 03-authentication.md
│   ├── 04-authorization.md
│   ├── 05-tenant-isolation.md
│   ├── 06-compliance.md
│   └── 07-incident-response.md
│
├── 10-api-contracts/                   # OpenAPI, API standards, versioning
│   ├── README.md
│   ├── 01-openapi-spec.md
│   ├── 02-api-standards.md
│   ├── 03-versioning.md
│   ├── 04-error-handling.md
│   ├── 05-pagination-filtering.md
│   ├── 06-authentication.md
│   ├── 07-rate-limiting.md
│   └── 08-codegen.md
│
├── 11-database/                        # Data model, migrations, RLS, performance
│   ├── README.md
│   ├── 01-data-model.md
│   ├── 02-migrations.md
│   ├── 03-rls-policies.md
│   ├── 04-indexing.md
│   ├── 05-retention.md
│   └── 06-backup.md
│
├── 12-design-system/                   # UI/UX tokens, components, patterns
│   ├── README.md
│   ├── 01-tokens.md
│   ├── 02-components.md
│   ├── 03-patterns.md
│   ├── 04-accessibility.md
│   ├── 05-theming.md
│   └── 06-rtl-ltr.md
│
├── 13-devops-cicd/                     # CI/CD pipelines, Jenkins, GitHub Actions
│   ├── README.md
│   ├── 01-jenkins.md
│   ├── 02-github-actions.md
│   ├── 03-quality-gates.md
│   ├── 04-release-automation.md
│   └── 05-rollback.md
│
├── 14-observability/                   # Metrics, logs, traces, alerting
│   ├── README.md
│   ├── 01-metrics.md
│   ├── 02-logging.md
│   ├── 03-tracing.md
│   ├── 04-alerting.md
│   ├── 05-slo-sli.md
│   └── 06-dashboards.md
│
├── 15-disaster-recovery/               # DR plans, RTO/RPO, failover
│   ├── README.md
│   ├── 01-dr-plan.md
│   ├── 02-rto-rpo.md
│   ├── 03-failover.md
│   └── 04-testing.md
│
├── 16-performance-scalability/         # Performance budgets, scaling strategies
│   ├── README.md
│   ├── 01-performance-budgets.md
│   ├── 02-caching.md
│   ├── 03-database-scaling.md
│   └── 04-load-testing.md
│
├── 17-integrations/                    # External integrations (Lansweeper, etc.)
│   ├── README.md
│   ├── 01-lansweeper.md
│   ├── 02-email-sms.md
│   ├── 03-webhooks.md
│   └── 04-third-party.md
│
├── 18-mobile/                          # Mobile architecture, offline, sync
│   ├── README.md
│   ├── 01-architecture.md
│   ├── 02-offline-sync.md
│   ├── 03-push-notifications.md
│   └── 04-app-store.md
│
├── 19-user-guides/                     # End-user documentation
│   ├── README.md
│   ├── 01-getting-started.md
│   ├── 02-assets.md
│   ├── 03-lansweeper.md
│   ├── 04-reports.md
│   └── 05-admin.md
│
├── 20-adr-index/                       # Architecture Decision Records index
│   ├── README.md
│   └── adr-log.md
│
├── 21-diagrams/                        # Architecture diagrams (C4, sequence, etc.)
│   ├── README.md
│   ├── c4-context.mmd
│   ├── c4-container.mmd
│   ├── c4-component.mmd
│   └── sequences/
│
└── 22-appendices/                      # Legacy, references, distribution logs
    ├── README.md
    ├── 01-legacy-fundamentals.md
    ├── 02-distribution-manifest.md
    ├── 03-references.md
    └── 04-arabic-summary.md
```

---

## 4. Execution Prompts (`prompts/`)

```text
prompts/
├── README.md                           # Prompt pack overview & usage
├── 00-global-context.md                # Global rules, naming, conventions
├── 01-foundation.md                    # Phase 01: Foundation & tooling
├── 02-domain-analysis.md               # Phase 02: Domain analysis
├── 03-backend-clean-architecture.md    # Phase 03: Backend Clean Architecture
├── 04-database-multitenancy.md         # Phase 04: Database & Multi-tenancy
├── 05-api-openapi.md                   # Phase 05: API & OpenAPI contracts
├── 06-lansweeper-integration.md        # Phase 06: Lansweeper integration
├── 07-frontend-nextjs.md               # Phase 07: Frontend (Next.js/BFF)
├── 08-design-system.md                 # Phase 08: Design system
├── 09-mobile.md                        # Phase 09: Mobile (Android baseline)
├── 10-event-driven-architecture.md     # Phase 10: Event-driven architecture
├── 11-testing-quality.md               # Phase 11: Testing & quality gates
├── 12-security.md                      # Phase 12: Security & compliance
├── 13-devops-infrastructure.md         # Phase 13: DevOps & infrastructure
├── 14-cicd-automation.md               # Phase 14: CI/CD automation
├── 15-observability.md                 # Phase 15: Observability
├── 16-backup-disaster-recovery.md      # Phase 16: Backup & DR
├── 17-performance-optimization.md      # Phase 17: Performance
├── 18-documentation-adr.md             # Phase 18: Documentation & ADRs
├── 19-release-deployment.md            # Phase 19: Release & deployment
└── 20-final-audit.md                   # Phase 20: Final audit & verification
```

> **Note** — Each prompt is designed for a fresh AI session. Run phases in order; verify before advancing.

---

## 5. Platform (`platform/`) — Core Implementation

### 5.1 Backend (`platform/backend/`)

```text
platform/backend/
├── manage.py                           # Django management script
├── pyproject.toml                      # Backend Python project config
├── uv.lock                             # Python lockfile
│
├── config/                             # Django project package (nexusasset/)
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py                     # Base settings
│   │   ├── local.py                    # Local development
│   │   ├── test.py                     # Test settings
│   │   ├── staging.py                  # Staging settings
│   │   └── production.py               # Production settings
│   ├── urls.py                         # Root URL configuration
│   ├── asgi.py                         # ASGI entrypoint
│   ├── wsgi.py                         # WSGI entrypoint
│   ├── celery.py                       # Celery app configuration
│   └── logging.py                      # Logging configuration
│
├── contexts/                           # Bounded contexts (Clean Architecture)
│   ├── core/                           # nxa_core — Shared kernel, cross-cutting
│   │   ├── README.md
│   │   ├── domain/                     # Entities, VOs, events, domain services
│   │   ├── application/                # Use cases, DTOs, ports
│   │   ├── infrastructure/             # Shared adapters, utilities
│   │   ├── interfaces/                 # Internal interfaces
│   │   ├── migrations/
│   │   └── tests/
│   │       ├── unit/
│   │       ├── integration/
│   │       └── api/
│   │
│   ├── accounts/                       # nxa_accounts — Identity, auth, roles
│   │   ├── README.md
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   ├── interfaces/
│   │   ├── migrations/
│   │   └── tests/
│   │
│   ├── identity/                       # nxa_identity — Extended identity (optional)
│   ├── tenants/                        # nxa_tenants — Tenancy, membership, hierarchy
│   ├── organizations/                  # nxa_organizations — Org structure (optional)
│   │
│   ├── assets/                         # nxa_assets — CMDB, asset lifecycle
│   │   ├── README.md
│   │   ├── domain/
│   │   ├── application/
│   │   ├── infrastructure/
│   │   ├── interfaces/
│   │   ├── migrations/
│   │   └── tests/
│   │
│   ├── asset_lifecycle/                # nxa_asset_lifecycle — Advanced lifecycle (optional)
│   │
│   ├── lansweeper/                     # nxa_lansweeper — Discovery sync (CRITICAL)
│   │   ├── README.md
│   │   ├── domain/
│   │   │   ├── sync_policy.py          # Field ownership, conflict resolution
│   │   │   ├── discovery.py            # Discovery aggregates
│   │   │   └── events.py               # Domain events
│   │   ├── application/
│   │   │   ├── sync_usecase.py         # Full/delta/bulk/webhook orchestration
│   │   │   ├── mapping_service.py      # Normalization, transformation
│   │   │   └── conflict_service.py     # Conflict detection, queue
│   │   ├── infrastructure/
│   │   │   ├── graphql_client.py       # Lansweeper GraphQL client
│   │   │   ├── mappers/                # Domain ↔ External mappers
│   │   │   ├── job_store.py            # Sync cursor, job persistence
│   │   │   └── webhook_verifier.py     # HMAC verification
│   │   ├── interfaces/
│   │   │   ├── api/                    # REST endpoints
│   │   │   ├── webhooks/               # Webhook receivers
│   │   │   └── tasks/                  # Celery task entrypoints
│   │   ├── migrations/
│   │   └── tests/
│   │       ├── unit/
│   │       ├── integration/
│   │       ├── api/
│   │       └── contract/
│   │
│   ├── integrations/                   # nxa_integrations — Generic integration framework
│   ├── billing/                        # nxa_billing — Subscriptions, invoicing
│   ├── subscriptions/                  # nxa_subscriptions — Entitlements, plans
│   ├── notifications/                  # nxa_notifications — Multi-channel delivery
│   ├── audit/                          # nxa_audit — Immutable audit trail
│   ├── reporting/                      # nxa_reporting — Analytics, exports
│   └── compliance/                     # nxa_compliance — Policy, attestation
│
├── shared/                             # Cross-context shared kernel
│   ├── kernel/                         # Base classes, primitives
│   ├── exceptions/                     # Domain exceptions
│   ├── events/                         # Event bus, publishers
│   ├── permissions/                    # Permission framework
│   ├── tenant_context/                 # Tenant resolution, middleware
│   ├── observability/                  # Metrics, tracing, logging helpers
│   └── utilities/                      # Common utilities
│
└── tests/                              # Cross-context test suites
    ├── unit/
    ├── integration/
    ├── security/
    ├── architecture/                   # import-linter, arch tests
    └── factories/                      # Test data factories
```

### 5.2 Frontend (`platform/frontend/`)

```text
platform/frontend/
├── next.config.ts                      # Next.js configuration
├── tsconfig.json                       # TypeScript configuration
├── eslint.config.mjs                   # ESLint flat config
├── components.json                     # shadcn/ui configuration
├── package.json                        # Frontend dependencies
├── pnpm-lock.yaml                      # pnpm lockfile
├── .env.example                        # Frontend env template
│
├── public/                             # Static assets
│   ├── favicon.ico
│   ├── robots.txt
│   └── manifest.json
│
├── src/
│   ├── app/                            # App Router (Next.js 15)
│   │   ├── (public)/                   # Public route group
│   │   │   ├── login/
│   │   │   ├── register/
│   │   │   └── layout.tsx
│   │   ├── (auth)/                     # Authenticated route group
│   │   │   ├── layout.tsx              # Auth layout with sidebar/header
│   │   │   ├── dashboard/
│   │   │   ├── assets/
│   │   │   ├── tenants/
│   │   │   ├── organizations/
│   │   │   ├── lansweeper/
│   │   │   ├── integrations/
│   │   │   ├── billing/
│   │   │   ├── subscriptions/
│   │   │   ├── audit/
│   │   │   ├── compliance/
│   │   │   ├── reports/
│   │   │   ├── notifications/
│   │   │   ├── settings/
│   │   │   └── profile/
│   │   │
│   │   ├── api/                        # BFF API routes
│   │   │   ├── auth/                   # Auth proxy, token refresh
│   │   │   ├── proxy/                  # Backend API proxy
│   │   │   └── health/                 # Health checks
│   │   │
│   │   ├── layout.tsx                  # Root layout
│   │   ├── page.tsx                    # Landing / redirect
│   │   ├── globals.css                 # Global styles (Tailwind)
│   │   ├── error.tsx                   # Global error boundary
│   │   ├── not-found.tsx               # 404 page
│   │   └── loading.tsx                 # Global loading
│   │
│   ├── proxy.ts                        # BFF proxy configuration
│   │
│   ├── components/                     # Shared UI components
│   │   ├── ui/                         # shadcn/ui primitives
│   │   ├── layout/                     # Layout components (Sidebar, Header)
│   │   ├── forms/                      # Form components
│   │   ├── tables/                     # Data tables, grids
│   │   ├── charts/                     # Visualization components
│   │   ├── feedback/                   # Toasts, modals, alerts
│   │   ├── navigation/                 # Breadcrumbs, tabs, pagination
│   │   └── providers/                  # Context providers
│   │
│   ├── features/                       # Feature-specific components
│   │   ├── auth/
│   │   ├── assets/
│   │   ├── tenants/
│   │   ├── organizations/
│   │   ├── lansweeper/
│   │   ├── integrations/
│   │   ├── billing/
│   │   ├── audit/
│   │   ├── compliance/
│   │   └── reporting/
│   │
│   ├── lib/                            # Core libraries
│   │   ├── api/                        # API client (generated from OpenAPI)
│   │   ├── auth/                       # Auth utilities
│   │   ├── permissions/                # Permission helpers
│   │   ├── validation/                 # Zod schemas
│   │   ├── i18n/                       # Internationalization
│   │   ├── telemetry/                  # Analytics, error tracking
│   │   └── utilities/                  # Common utilities
│   │
│   ├── hooks/                          # Custom React hooks
│   ├── stores/                         # State management (Zustand/Jotai)
│   ├── types/                          # Shared TypeScript types
│   └── styles/                         # Additional styles
│
└── tests/                              # Frontend tests
    ├── unit/
    ├── integration/
    ├── e2e/
    └── visual/
```

### 5.3 Mobile (`platform/mobile/`)

```text
platform/mobile/
├── android/                            # Android (Kotlin + Jetpack Compose)
│   ├── app/
│   │   ├── src/
│   │   │   ├── main/
│   │   │   │   ├── java/com/nexusasset/cloud/
│   │   │   │   │   ├── ui/             # Compose UI
│   │   │   │   │   ├── data/           # Repository, datasources
│   │   │   │   │   ├── domain/         # Use cases, entities
│   │   │   │   │   ├── di/             # Hilt modules
│   │   │   │   │   └── MainApplication.kt
│   │   │   │   └── AndroidManifest.xml
│   │   │   ├── test/                   # Unit tests
│   │   │   └── androidTest/            # Instrumented tests
│   │   ├── build.gradle.kts
│   │   └── proguard-rules.pro
│   ├── gradle/
│   ├── build.gradle.kts
│   ├── settings.gradle.kts
│   └── gradle.properties
│
├── ios/                                # iOS (Swift + SwiftUI) — Future
│   └── README.md
│
├── shared-contracts/                   # Shared types, schemas (Kotlin/Swift/TS)
│   └── README.md
│
└── docs/                               # Mobile-specific docs
    ├── architecture.md
    ├── offline-sync.md
    └── push-notifications.md
```

### 5.4 Shared Packages (`platform/packages/@nexusasset/`)

```text
platform/packages/@nexusasset/
├── api-client/                         # Generated API client (TypeScript)
│   ├── package.json
│   ├── src/
│   │   ├── client.ts
│   │   ├── schemas/
│   │   └── hooks/
│   └── README.md
│
├── shared-types/                       # Shared TypeScript types
│   ├── package.json
│   ├── src/
│   │   ├── api/
│   │   ├── domain/
│   │   └── common/
│   └── README.md
│
├── design-tokens/                      # Design tokens (JSON + TS)
│   ├── package.json
│   ├── tokens/
│   │   ├── colors.json
│   │   ├── typography.json
│   │   ├── spacing.json
│   │   ├── sizing.json
│   │   ├── radius.json
│   │   ├── shadows.json
│   │   ├── breakpoints.json
│   │   ├── z-index.json
│   │   └── motion.json
│   ├── src/
│   └── README.md
│
├── ui/                                 # Shared UI components (React)
│   ├── package.json
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── utils/
│   └── README.md
│
├── eslint-config/                      # Shared ESLint config
│   ├── package.json
│   ├── index.js
│   └── README.md
│
├── tsconfig/                           # Shared TypeScript configs
│   ├── package.json
│   ├── base.json
│   ├── nextjs.json
│   ├── react-library.json
│   └── node.json
│
└── contracts/
    └── openapi.yaml.description.md     # OpenAPI 3.1 contract descriptor
```

### 5.5 Contracts (`platform/contracts/`)

```text
platform/contracts/
├── openapi/                            # OpenAPI 3.1 Specification
│   ├── openapi.yaml                    # Main OpenAPI document (SOURCE OF TRUTH)
│   ├── schemas/                        # Reusable schema components
│   │   ├── common.yaml
│   │   ├── assets.yaml
│   │   ├── tenants.yaml
│   │   ├── lansweeper.yaml
│   │   ├── billing.yaml
│   │   └── audit.yaml
│   ├── paths/                          # Path definitions by bounded context
│   │   ├── assets.yaml
│   │   ├── tenants.yaml
│   │   ├── lansweeper.yaml
│   │   ├── billing.yaml
│   │   └── audit.yaml
│   └── versions/                       # Versioned specs
│       └── v1/
│
├── asyncapi/                           # AsyncAPI for event-driven contracts
│   └── asyncapi.yaml
│
├── events/                             # Event definitions
│   ├── asset-events.yaml
│   ├── tenant-events.yaml
│   └── integration-events.yaml
│
├── webhooks/                           # Webhook payload schemas
│   ├── lansweeper.yaml
│   └── billing.yaml
│
└── generated/                          # Codegen outputs (gitignored in source)
    ├── typescript/
    ├── kotlin/
    └── swift/
```

### 5.6 Design System (`platform/design-system/`)

```text
platform/design-system/
├── tokens/                             # Design tokens (source of truth)
│   ├── colors.json
│   ├── typography.json
│   ├── spacing.json
│   ├── sizing.json
│   ├── radius.json
│   ├── shadows.json
│   ├── breakpoints.json
│   ├── z-index.json
│   └── motion.json
│
├── assets/                             # Static design assets
│   ├── fonts/
│   ├── icons/
│   ├── images/
│   └── illustrations/
│
├── components/                         # Component library (Storybook)
│   ├── package.json
│   ├── src/
│   │   ├── primitives/
│   │   ├── composites/
│   │   └── patterns/
│   ├── patterns/
│   └── .storybook/
│
├── themes/                             # Theme definitions
│   ├── light/
│   ├── dark/
│   └── high-contrast/
│
├── storybook/                          # Storybook configuration
│   ├── main.ts
│   ├── preview.ts
│   └── theme.ts
│
└── docs/                               # Design system documentation
    ├── tokens.md
    ├── components.md
    ├── patterns.md
    ├── accessibility.md
    ├── rtl-ltr.md
    ├── theming.md
    └── ai-coding-rules.md
```

### 5.7 Infrastructure (`platform/infrastructure/`)

```text
platform/infrastructure/
├── docker/                             # Dockerfiles for each service
│   ├── backend.Dockerfile
│   ├── frontend.Dockerfile
│   ├── worker.Dockerfile
│   ├── scheduler.Dockerfile
│   └── security-scanning.Dockerfile
│
├── traefik/                            # Traefik v3 edge configuration
│   ├── traefik.yml                     # Static configuration
│   ├── dynamic/                        # Dynamic configuration (labels, middleware)
│   │   ├── middlewares.yml
│   │   ├── routes.yml
│   │   └── tls.yml
│   ├── tls/                            # TLS certificates (Let's Encrypt)
│   └── middlewares/                    # Custom middleware configs
│
├── postgres/                           # PostgreSQL 16 configuration
│   ├── config/                         # postgresql.conf, pg_hba.conf
│   ├── init/                           # Initialization scripts
│   ├── extensions/                     # Required extensions
│   ├── rls/                            # RLS policy definitions
│   ├── roles/                          # Role definitions
│   ├── migrations/                     # Schema migrations (Flyway/Liquibase)
│   └── backup/                         # Backup scripts, configs
│
├── redis/                              # Redis 7 configuration
│   ├── config/                         # redis.conf
│   └── policies/                       # Eviction, memory policies
│
├── rabbitmq/                           # RabbitMQ (optional, for async)
│   └── config/
│
├── storage/                            # Object storage (S3-compatible)
│   └── object-storage/
│
└── scripts/                            # Container utility scripts
    ├── health-check.sh
    ├── wait-for-dependencies.sh
    └── container-entrypoint.sh
```

### 5.8 Deployment (`platform/deployment/`)

```text
platform/deployment/
├── ansible/                            # Ansible automation
│   ├── ansible.cfg
│   ├── inventories/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   ├── playbooks/
│   │   ├── bootstrap.yml               # Initial server setup
│   │   ├── hardening.yml               # Security hardening
│   │   ├── deploy.yml                  # Application deployment
│   │   ├── upgrade.yml                 # Version upgrade
│   │   ├── rollback.yml                # Rollback procedure
│   │   ├── backup.yml                  # Backup execution
│   │   └── restore.yml                 # Restore procedure
│   └── roles/
│       ├── common/
│       ├── docker/
│       ├── firewall/
│       ├── traefik/
│       ├── monitoring/
│       ├── application/
│       └── backup/
│
├── ci-cd/                              # CI/CD pipeline definitions
│   ├── Jenkinsfile                     # Jenkins declarative pipeline
│   ├── jenkins/
│   │   ├── casc/                       # Configuration as Code
│   │   ├── shared-library/             # Shared pipeline library
│   │   └── agents/                     # Agent configurations
│   ├── github/
│   │   └── workflows/                  # GitHub Actions workflows
│   ├── quality-gates/                  # Quality gate definitions
│   └── release/                        # Release automation
│
├── environments/                       # Environment-specific configs
│   ├── dev/
│   ├── staging/
│   └── production/
│
└── release-management/                 # Release process docs
    ├── versioning.md
    ├── release-checklist.md
    ├── rollback-policy.md
    └── migration-policy.md
```

### 5.9 Security (`platform/security/`)

```text
platform/security/
├── threat-model/                       # Threat modeling artifacts
│   ├── data-flow-diagrams/
│   ├── threat-lists/
│   └── mitigations/
│
├── policies/                           # Security policies
│   ├── security-policy.md
│   ├── vulnerability-management.md
│   └── secure-coding.md
│
├── iam/                                # Identity & Access Management
│   ├── roles/
│   ├── policies/
│   └── service-accounts/
│
├── secrets-management/                 # Secrets handling
│   ├── sops/                           # SOPS encrypted secrets
│   └── templates/                      # Secret templates
│
├── encryption/                         # Encryption configurations
│   ├── at-rest/
│   ├── in-transit/
│   └── key-management/
│
├── certificates/                       # Certificate management
│   ├── ca/
│   ├── server/
│   └── client/
│
├── access-control/                     # Access control configurations
│   ├── rbac/
│   ├── abac/
│   └── network-policies/
│
├── tenant-isolation/                   # Tenant isolation verification
│   ├── rls-tests/
│   ├── network-segmentation/
│   └── data-access-tests/
│
├── supply-chain/                       # Supply chain security
│   ├── sbom/                           # Software Bill of Materials
│   ├── image-signing/                  # Cosign/Notary
│   └── provenance/                     # SLSA provenance
│
├── scans/                              # Security scan configurations
│   ├── sast/
│   ├── dast/
│   ├── dependency/
│   ├── container/
│   └── secrets/
│
├── compliance/                         # Compliance evidence
│   ├── soc2/
│   ├── iso27001/
│   ├── gdpr/
│   └── pci-dss/
│
├── owasp/                              # OWASP mappings
│   ├── top10/
│   ├── asvs/
│   └── masvs/
│
└── incident-response/                  # Incident response procedures
    ├── playbooks/
    ├── runbooks/
    └── communication/
```

### 5.10 Observability (`platform/observability/`)

```text
platform/observability/
├── prometheus/                         # Prometheus configuration
│   ├── prometheus.yml
│   ├── rules/
│   └── alerts/
│
├── grafana/                            # Grafana dashboards & provisioning
│   ├── dashboards/
│   │   ├── overview.json
│   │   ├── backend/
│   │   ├── frontend/
│   │   ├── database/
│   │   ├── cache/
│   │   ├── lansweeper/
│   │   └── business/
│   └── provisioning/
│       ├── datasources/
│       └── dashboards/
│
├── loki/                               # Loki log aggregation
│   ├── loki.yml
│   └── rules/
│
├── tempo/                              # Tempo distributed tracing
│   ├── tempo.yml
│   └── traces/
│
├── alertmanager/                       # Alertmanager configuration
│   ├── alertmanager.yml
│   ├── templates/
│   └── receivers/
│
├── otel/                               # OpenTelemetry Collector
│   ├── collector/
│   │   ├── config.yaml
│   │   └── pipelines/
│   ├── traces/
│   └── metrics/
│
├── alert-rules/                        # Unified alert rules
│   ├── infrastructure.yaml
│   ├── application.yaml
│   ├── business.yaml
│   └── security.yaml
│
├── slo-sli/                            # Service Level Objectives/Indicators
│   ├── definitions/
│   ├── dashboards/
│   └── burn-rate-alerts/
│
└── synthetic-monitoring/               # Synthetic checks
    ├── api-checks/
    ├── ui-checks/
    └── ssl-checks/
```

### 5.11 Operations (`platform/ops/`)

```text
platform/ops/
├── runbooks/                           # Operational runbooks
│   ├── 01-deployment.md
│   ├── 02-rollback.md
│   ├── 03-database-migration.md
│   ├── 04-cache-invalidation.md
│   ├── 05-lansweeper-sync.md
│   ├── 06-certificate-renewal.md
│   ├── 07-incident-response.md
│   ├── 08-capacity-scaling.md
│   └── 09-disaster-recovery.md
│
├── incident-management/                # Incident management process
│   ├── severity-levels.md
│   ├── escalation.md
│   ├── postmortem-template.md
│   └── communication.md
│
├── backup-restore/                     # Backup & restore procedures
│   ├── database-backup.md
│   ├── database-restore.md
│   ├── object-storage-backup.md
│   └── verification.md
│
├── disaster-recovery/                  # DR procedures
│   ├── dr-plan.md
│   ├── failover.md
│   ├── failback.md
│   └── testing.md
│
├── capacity-planning/                  # Capacity planning
│   ├── metrics.md
│   ├── forecasting.md
│   └── scaling-triggers.md
│
├── maintenance/                        # Maintenance procedures
│   ├── scheduled-maintenance.md
│   ├── patch-management.md
│   └── dependency-updates.md
│
├── database-operations/                # Database operations
│   ├── migrations.md
│   ├── vacuum-analyze.md
│   ├── index-maintenance.md
│   └── replication.md
│
├── certificate-renewal/                # TLS certificate renewal
│   ├── letsencrypt.md
│   ├── monitoring.md
│   └── troubleshooting.md
│
├── dependency-updates/                 # Dependency update process
│   ├── schedule.md
│   ├── testing.md
│   └── rollback.md
│
└── service-catalog/                    # Service catalog
    ├── services.yaml
    ├── owners.yaml
    └── slos.yaml
```

### 5.12 Tests (`platform/tests/`)

```text
platform/tests/
├── architecture/                       # Architecture verification tests
│   ├── import_linter.yml
│   ├── boundary_tests.py
│   └── layer_tests.py
│
├── contract/                           # Contract testing (Pact/Schemathesis)
│   ├── provider/
│   ├── consumer/
│   └── schemas/
│
├── api/                                # API integration tests
│   ├── assets/
│   ├── tenants/
│   ├── lansweeper/
│   ├── billing/
│   └── audit/
│
├── integration/                        # Cross-service integration tests
│   ├── backend-frontend/
│   ├── backend-worker/
│   └── external/
│
├── e2e/                                # End-to-end tests (Playwright/Cypress)
│   ├── auth/
│   ├── assets/
│   ├── lansweeper/
│   ├── billing/
│   └── admin/
│
├── security/                           # Security tests
│   ├── authentication/
│   ├── authorization/
│   ├── tenant-isolation/
│   ├── injection/
│   └── secrets/
│
├── performance/                        # Performance benchmarks
│   ├── api/
│   ├── database/
│   └── sync/
│
├── load/                               # Load testing (k6/Locust)
│   ├── scenarios/
│   ├── thresholds/
│   └── reports/
│
├── resilience/                         # Chaos engineering, resilience
│   ├── experiments/
│   └── game-days/
│
├── accessibility/                      # A11y tests (axe-core)
│   ├── components/
│   └── pages/
│
├── migration/                          # Migration testing
│   ├── schema/
│   └── data/
│
└── disaster-recovery/                  # DR testing
    ├── scenarios/
    └── results/
```

### 5.13 Quality (`platform/quality/`)

```text
platform/quality/
├── lint/                               # Linting configurations
│   ├── python/
│   ├── typescript/
│   ├── docker/
│   └── yaml/
│
├── type-check/                         # Type checking configs
│   ├── python/                         # mypy/pyright
│   └── typescript/                     # tsc
│
├── coverage/                           # Coverage thresholds
│   ├── backend.yml
│   ├── frontend.yml
│   └── mobile.yml
│
├── security/                           # Security quality gates
│   ├── bandit.yml
│   ├── safety.yml
│   ├── trivy.yml
│   └── semgrep.yml
│
├── performance-budgets/                # Performance budgets
│   ├── frontend.json
│   ├── api.json
│   └── database.json
│
├── dependency-audit/                   # Dependency audit configs
│   ├── python/
│   ├── node/
│   └── container/
│
├── code-quality/                       # Code quality metrics
│   ├── sonarcloud.yml
│   ├── complexity.yml
│   └── duplication.yml
│
└── quality-gate.sh                     # Unified quality gate script
```

### 5.14 Scripts (`platform/scripts/`)

```text
platform/scripts/
├── bootstrap.sh                        # Full environment bootstrap
├── setup-dev.sh                        # Development environment setup
├── generate-contracts.sh               # Generate API clients from OpenAPI
├── generate-tokens.sh                  # Generate design token outputs
├── database-migrate.sh                 # Run database migrations
├── database-backup.sh                  # Backup database
├── database-restore.sh                 # Restore database
├── seed.py                             # Seed development data
├── release.sh                          # Release automation
├── health-check.sh                     # System health check
├── security-scan.sh                    # Run security scans
└── quality-gate.sh                     # Run all quality gates
```

---

## 6. Project Metadata (`project-meta/`)

```text
project-meta/
├── governance/                         # Governance documents
│   ├── project-charter.md
│   ├── engineering-principles.md
│   ├── architecture-governance.md
│   ├── coding-standards.md
│   ├── documentation-standards.md
│   ├── change-management.md
│   ├── release-governance.md
│   └── quality-gates.md
│
├── policies/                           # Organizational policies
│   ├── security-policy.md
│   ├── privacy-policy.md
│   ├── data-classification.md
│   ├── retention-policy.md
│   ├── backup-policy.md
│   ├── dependency-policy.md
│   ├── access-control-policy.md
│   └── tenant-isolation-policy.md
│
├── adr/                                # Architecture Decision Records
│   ├── README.md
│   └── templates/
│       └── adr-template.md
│
├── standards/                          # Technical standards
│   ├── api-standards.md
│   ├── database-standards.md
│   ├── frontend-standards.md
│   ├── backend-standards.md
│   ├── devops-standards.md
│   └── testing-standards.md
│
├── runtime/                            # Runtime descriptors
│   ├── supported-versions.md
│   ├── compatibility-matrix.md
│   └── dependency-lifecycle.md
│
└── templates/                          # Document templates
    ├── adr-template.md
    ├── feature-template.md
    ├── incident-template.md
    └── release-template.md
```

---

## 7. Archive (`archive/`)

```text
archive/
├── inputs/                             # Original input documents
├── legacy/                             # Legacy/migrated documents
├── snapshots/                          # Point-in-time snapshots
└── archive-index.md                    # Archive catalog & traceability
```

---

## 8. Naming & Placement Rules

┌─────────────────────────────────────────────────────────────────────┐
│ ▸ CANONICAL NAMING TABLE                                             │
├──────────────────────┬──────────────────────────────┬────────────────┤
│ Item                 │ Canonical Name               │ Placement      │
├──────────────────────┼──────────────────────────────┼────────────────┤
│ Repository root      │ `nexusasset-cloud/`          │ Root           │
│ Django project pkg   │ `nexusasset/`                │ `backend/config/`│
│ Bounded context apps │ `nxa_core`, `nxa_accounts`,  │ `platform/      │
│                      │ `nxa_tenants`, `nxa_assets`, │ `backend/       │
│                      │ `nxa_lansweeper`,            │ `contexts/`    │
│                      │ `nxa_billing`,               │                │
│                      │ `nxa_notifications`,         │                │
│                      │ `nxa_audit`                  │                │
│ Runtime services     │ `nxa-backend`, `nxa-frontend`,│ Docker/compose │
│                      │ `nxa-worker`, `nxa-db`,      │ infra configs  │
│                      │ `nxa-redis`                  │                │
│ Edge naming          │ `nxa-traefik-public`,        │ Traefik config │
│                      │ `nxa-le`                     │                │
│ Domains              │ `app.nexusasset.cloud`,      │ DNS/Traefik    │
│                      │ `api.nexusasset.cloud`       │ routing        │
│ Shared packages      │ `@nexusasset/*`              │ `platform/     │
│                      │                              │ packages/      │
│                      │                              │ @nexusasset/   │
│ Secret keys/env      │ `NXA_SECRET_KEY`, `NXA_*`    │ `.env`/secret  │
│                      │                              │ store (never   │
│                      │                              │ committed)     │
│ OpenAPI contract     │ `openapi.yaml`               │ `platform/     │
│                      │                              │ contracts/     │
│                      │                              │ openapi/`      │
└──────────────────────┴──────────────────────────────┴────────────────┘

### 8.1 Directory Naming Conventions

| Pattern | Convention | Example |
|---|---|---|
| Directories | kebab-case | `nxa-lansweeper`, `design-system` |
| Python packages/modules | snake_case | `nxa_accounts`, `sync_policy` |
| TypeScript packages | kebab-case | `@nexusasset/api-client` |
| Docker images | `nexusasset/<service>:<tag>` | `nexusasset/backend:v1.2.3` |
| Kubernetes resources | kebab-case | `nxa-backend-deployment` |
| Git branches | `type/scope-description` | `feat/assets-bulk-import` |

---

## 9. Validation Checklist

┌─────────────────────────────────────────────────────────────────────┐
│ ▸ STRUCTURE VALIDATION                                               │
├─────────────────────────────────────────────────────────────────────┤
│ ✅ Repository root is `nexusasset-cloud/`                            │
│ ✅ Django project package at `platform/backend/config/` (`nexusasset/`)│
│ ✅ All bounded contexts use `nxa_` prefix                            │
│ ✅ Each bounded context has `domain/application/infrastructure/      │
│    interfaces/tests/`                                                │
│ ✅ Service names use `nxa-*` in Docker/infrastructure configs        │
│ ✅ OpenAPI contract at `platform/contracts/openapi/openapi.yaml`    │
│ ✅ Shared packages under `platform/packages/@nexusasset/`            │
│ ✅ Infrastructure as code under `platform/infrastructure/`           │
│ ✅ Deployment automation under `platform/deployment/`                │
│ ✅ Documentation in `docs/` (00–22 structure)                        │
│ ✅ Execution prompts in `prompts/` (00–20)                           │
│ ✅ Governance/policies in `project-meta/`                            │
│ ✅ Archive in `archive/` with index                                  │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ CLEAN ARCHITECTURE VALIDATION                                      │
├─────────────────────────────────────────────────────────────────────┤
│ ✅ `domain` layer: NO Django/DRF/ORM/HTTP/Celery/UI imports         │
│ ✅ `application` layer: NO concrete ORM/framework details           │
│ ✅ `infrastructure` layer: Implements ports, does NOT own rules     │
│ ✅ `interfaces` layer: Translates HTTP/tasks → application          │
│ ✅ Dependency direction: Interfaces → Application → Domain          │
│ ✅ Import-linter configured and passing in CI                        │
│ ✅ Architecture tests in `platform/tests/architecture/`              │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ MULTI-TENANCY VALIDATION                                           │
├─────────────────────────────────────────────────────────────────────┤
│ ✅ All tenant tables include `tenant_id`                             │
│ ✅ RLS policies defined in `platform/infrastructure/postgres/rls/`  │
│ ✅ Tenant context middleware in `platform/backend/contexts/          │
│    shared/tenant_context/`                                           │
│ ✅ Cross-tenant negative tests in `platform/tests/security/`         │
│ ✅ Tenant isolation policy in `project-meta/policies/`               │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ LANSWEeper INTEGRATION VALIDATION                                  │
├─────────────────────────────────────────────────────────────────────┤
│ ✅ Field Ownership Matrix in `nxa_lansweeper/domain/sync_policy.py` │
│ ✅ Idempotent upsert with `source_id` + `source_system` constraint  │
│ ✅ Sync modes: full, delta, bulk, webhook implemented               │
│ ✅ Conflict detection & queue in `conflict_service.py`              │
│ ✅ HMAC webhook verification in `webhook_verifier.py`               │
│ ✅ Audit trail for every sync operation                             │
├─────────────────────────────────────────────────────────────────────┤
│ ▸ DOCUMENTATION VALIDATION                                           │
├─────────────────────────────────────────────────────────────────────┤
│ ✅ All docs follow NexusAsset Cloud format (header, ToC, sections)  │
│ ✅ Arabic executive summaries where required                        │
│ ✅ ADRs for all architectural decisions                             │
│ ✅ Runbooks for top 10 operational scenarios                        │
│ ✅ Cross-references use relative links                              │
└─────────────────────────────────────────────────────────────────────┘

---

## Appendix — Quick Reference: Directory Purpose Map

| Directory | Purpose | Owner |
|---|---|---|
| `docs/` | Enterprise documentation, ADRs, guides | Architecture/Tech Writers |
| `prompts/` | Phased execution prompts for AI sessions | Platform Engineering |
| `platform/backend/` | Django 5 + DRF Clean Architecture | Backend Team |
| `platform/frontend/` | Next.js 15 + React 19 + BFF | Frontend Team |
| `platform/mobile/` | Android (Kotlin), iOS (Swift) | Mobile Team |
| `platform/packages/` | Shared NPM packages (`@nexusasset/*`) | Platform Team |
| `platform/contracts/` | OpenAPI 3.1, AsyncAPI, Event schemas | API Guild |
| `platform/design-system/` | Tokens, components, Storybook | Design System Team |
| `platform/infrastructure/` | Docker, Traefik, Postgres, Redis configs | DevOps/Platform |
| `platform/deployment/` | Ansible, Jenkins, GitHub Actions, envs | DevOps |
| `platform/security/` | Threat model, policies, scans, compliance | Security |
| `platform/observability/` | Prometheus, Grafana, Loki, Tempo, OTel | SRE/Observability |
| `platform/ops/` | Runbooks, incident management, DR | SRE/Operations |
| `platform/tests/` | Architecture, contract, e2e, load, security | QA/Platform |
| `platform/quality/` | Lint, type-check, coverage, security gates | Platform |
| `platform/scripts/` | Bootstrap, migration, release, health | Platform |
| `project-meta/` | Governance, policies, ADRs, standards | Architecture/Leadership |
| `archive/` | Historical traceability | Platform |

---

**End of file — NexusAsset Cloud / docs@nexusasset.cloud**