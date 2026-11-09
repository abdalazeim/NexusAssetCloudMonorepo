╔════════════════════════════════════════════════════════════════════════════════════════════════╗
║ Document ID: DOC-05                                                                            ║
║ Title: Application Architecture — Clean Architecture Layers, Use Cases, Ports & Adapters       ║
║ Version: 1.0.0                                                                                 ║
║ Status: ✓ Approved                                                                             ║
║ Classification: Internal — Confidential                                                        ║
║ Authors: NexusAsset Cloud Architecture Team                                                    ║
║ Reviewers: Engineering Leads, Security, QA, Platform                                           ║
║ Approved By: CTO / Chief Architect / VP Engineering                                            ║
║ Date: 2026-11-09                                                                               ║
║ Supersedes: —                                                                                  ║
║ Related: DOC-00, DOC-01, DOC-02, DOC-03, DOC-04, DOC-06, DOC-07, DOC-10, DOC-11, DOC-13       ║
║ Traceability: UC-### ← REQ-FN-###, UC-### → Port/Adapter, ADR-### → Architecture Decisions    ║
╚════════════════════════════════════════════════════════════════════════════════════════════════╝

# ═══════════════════════════════════════════════════════════════
# NexusAsset Cloud — Application Architecture (DOC-05)
# ═══════════════════════════════════════════════════════════════

> **Executive Summary** — This document defines the application architecture for NexusAsset Cloud following **Clean Architecture** principles (Robert C. Martin) with **Domain-Driven Design** (Eric Evans) tactical patterns. The architecture enforces the **Dependency Rule**: inner layers (Domain, Application) have zero dependencies on outer layers (Infrastructure, Frameworks, UI). All business logic resides in the Domain layer (DOC-04); Application layer contains Use Cases orchestrating domain objects; Infrastructure layer implements Ports (interfaces) with Adapters (Django ORM, Kafka, Redis, HTTP clients). This document specifies layer boundaries, module structure, use case catalog, port contracts, adapter implementations, and cross-cutting concerns.

---

## Table of Contents

1. [Architecture Overview & Principles](#1-architecture-overview--principles)
2. [Layer Definitions & Boundaries](#2-layer-definitions--boundaries)
3. [Module Structure (Monorepo Layout)](#3-module-structure-monorepo-layout)
4. [Use Case Catalog per Context](#4-use-case-catalog-per-context)
5. [Port Contracts (Interfaces)](#5-port-contracts-interfaces)
6. [Adapter Implementations](#6-adapter-implementations)
6. [Cross-Cutting Concerns](#7-cross-cutting-concerns)
8. [Dependency Injection & Composition Root](#8-dependency-injection--composition-root)
9. [Transaction Management & Unit of Work](#9-transaction-management--unit-of-work)
10. [Event Publishing & Outbox Pattern](#10-event-publishing--outbox-pattern)
11. [Error Handling & Result Types](#11-error-handling--result-types)
12. [Architecture Decision Records (ADRs)](#12-architecture-decision-records-adrs)
13. [Diagrams](#13-diagrams)
14. [Tables](#14-tables)

---

## 1. Architecture Overview & Principles

### 1.1 Clean Architecture Layer Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    CLEAN ARCHITECTURE LAYERS                                        │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                              FRAMEWORKS & DRIVERS (Outer)                                   │   │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │   │
│   │  │   Django    │  │  Next.js    │  │  Celery     │  │   Kafka     │  │  PostgreSQL │      │   │
│   │  │   REST      │  │   React     │  │  Workers    │  │  Consumers  │  │  (ORM)      │      │   │
│   │  │   Framework │  │   (BFF)     │  │             │  │             │  │             │      │   │
│   │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘      │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                            │                                                        │
│                                            │ Implements                                             │
│                                            ▼                                                        │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                           INTERFACE ADAPTERS (Adapters)                                     │   │
│   │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────┐    │   │
│   │  │  Django ORM     │  │  Kafka Producer │  │  Redis Cache    │  │  HTTP Clients           │    │   │
│   │  │  Repositories   │  │  (Outbox)       │  │  Adapter        │  │  (Lansweeper, Stripe,   │    │   │
│   │  │                 │  │                 │  │                 │  │   Cloud Providers)      │    │   │
│   │  └─────────────────┘ └─────────────────┘ └─────────────────┘ └─────────────────────────┘    │   │
│   │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────┐    │   │
│   │  │  Django Auth    │  │  Metrics        │  │  File Storage   │  │  Email/SMS Providers    │    │   │
│   │  │  Adapter        │  │  Exporter       │  │  (S3/MinIO)     │  │  (SendGrid, Twilio)     │    │   │
│   │  └─────────────────┘ └─────────────────┘ └─────────────────┘ └─────────────────────────┘    │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                            │                                                        │
│                                            │ Implements                                             │
│                                            ▼                                                        │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                        APPLICATION LAYER (Use Cases)                                        │   │
│   │  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │   │
│   │  │ Use Case: RegisterAsset          │ Use Case: SyncLansweeperAssets                   │   │   │
│   │  │ Use Case: AuthorizeAsset         │ Use Case: GenerateInvoice                        │   │   │
│   │  │ Use Case: RunComplianceCheck     │ Use Case: ProvisionTenant                        │   │   │
│   │  │ Use Case: AssignAsset            │ Use Case: SendNotification                       │   │   │
│   │  │ ... (200+ use cases)             │ ...                                              │   │   │
│   │  └─────────────────────────────────────────────────────────────────────────────────────┘   │   │
│   │  • Orchestrates Domain Objects (Aggregates, Entities, VOs)                                │   │
│   │  • Coordinates Repositories (Ports)                                                        │   │
│   │  • Publishes Domain Events                                                                  │   │
│   │  • NO Django, NO ORM, NO HTTP, NO Infrastructure                                           │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                            │                                                        │
│                                            │ Uses                                                    │
│                                            ▼                                                        │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                           DOMAIN LAYER (Core)                                               │   │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │   │
│   │  │ Aggregates  │  │ Entities    │  │ Value Obj.  │  │ Domain      │  │ Domain      │      │   │
│   │  │ (Roots)     │  │ (Nested)    │  │ (Immutable) │  │ Events      │  │ Services    │      │   │
│   │  │             │  │             │  │             │  │ (EVT-###)   │  │ (DSV-###)   │      │   │
│   │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘      │   │
│   │  • Pure Python — ZERO external dependencies                                              │   │
│   │  • Invariants enforced in methods                                                          │   │
│   │  • Rich domain model (anemic forbidden)                                                    │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
│   DEPENDENCY RULE:  Inner layers know NOTHING about outer layers.                                 │
│   Dependencies point INWARD only.                                                                │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Architectural Principles

| Principle ID | Principle | Enforcement |
|--------------|-----------|-------------|
| **PRIN-001** | **Dependency Rule** — Inner layers never import outer layers | `import-linter` in CI; `pytest --import-linter` |
| **PRIN-002** | **Domain Purity** — Domain layer: zero third-party imports | `ruff` rule: `no-external-imports-in-domain` |
| **PRIN-003** | **Use Case per User Goal** — One use case class per UC-### from DOC-02 | Code review checklist; architecture tests |
| **PRIN-004** | **Ports Define Contracts** — Application defines interfaces; Infrastructure implements | Abstract base classes in `application/ports/` |
| **PRIN-005** | **Adapters Are Thin** — Adapters only translate; no business logic | Code review; max 50 lines per adapter method |
| **PRIN-006** | **Explicit Dependencies** — Use constructor injection; no service locator | `dependency-injector` container; wiring in `composition_root.py` |
| **PRIN-007** | **Transaction Boundaries at Use Case** — Use case manages Unit of Work | `@transactional` decorator on use case `execute()` |
| **PRIN-008** | **Events for Integration** — Cross-context via Integration Events (IEV-###) | Outbox pattern; event publishers in ports |

---

## 2. Layer Definitions & Boundaries

### 2.1 Domain Layer (`domain/`)

**Location**: `backend/nexusasset/<context>/domain/`

**Contents**:
- `aggregates/` — Aggregate roots (AGG-CTX-###)
- `entities/` — Nested entities (ENT-CTX-###)
- `value_objects/` — Immutable value objects (VO-CTX-###)
- `events/` — Domain events (EVT-CTX-###) as dataclasses
- `services/` — Domain services (DSV-###) — stateless, cross-aggregate logic
- `specifications/` — Reusable business rules (SPEC-###) — predicate objects
- `exceptions/` — Domain exceptions (e.g., `InsufficientQuotaError`, `InvalidStateTransitionError`)
- `ports/` — **Repository interfaces** (abstract base classes) — *owned by domain*

**Prohibited Imports**: `django`, `sqlalchemy`, `kafka`, `redis`, `requests`, `boto3`, `pydantic` (use `attrs`/`dataclasses`), `fastapi`, `celery`

**Allowed**: Python stdlib, `dataclasses`, `datetime`, `uuid`, `decimal`, `enum`, `abc`, `typing`, `dataclass-wizard` (for serialization only)

### 2.2 Application Layer (`application/`)

**Location**: `backend/nexusasset/<context>/application/`

**Contents**:
- `use_cases/` — Use case classes (one per UC-###)
- `dto/` — Data Transfer Objects (input/output) — `pydantic` models allowed here
- `ports/` — **Secondary ports** (interfaces for external systems): `EventPublisher`, `EmailSender`, `HttpClient`, `FileStorage`, `Cache`, `Metrics`
- `queries/` — Read-model queries (CQRS read side) — `pydantic` models for responses
- `mappers/` — Domain ↔ DTO mappers (pure functions)

**Allowed Imports**: Domain layer, `pydantic`, `typing`, `dataclasses`, stdlib

**Prohibited**: Django, ORM, Kafka, Redis, HTTP clients (use ports)

### 2.3 Infrastructure Layer (`infrastructure/`)

**Location**: `backend/nexusasset/<context>/infrastructure/`

**Contents**:
- `repositories/` — Repository implementations (Django ORM) implementing domain ports
- `django/` — Django models, migrations, admin, signals (persistence only)
- `messaging/` — Kafka producers/consumers, outbox processor
- `cache/` — Redis adapter implementing `CachePort`
- `http/` — HTTP clients (Lansweeper GraphQL, Stripe, Cloud Provider APIs)
- `storage/` — S3/MinIO adapter implementing `FileStoragePort`
- `auth/` — Django auth adapter implementing `AuthPort`
- `email/` — SendGrid/Twilio adapters implementing `EmailPort`, `SmsPort`
- `metrics/` — Prometheus exporter adapter
- `celery/` — Celery tasks (thin wrappers calling use cases)

**Allowed Imports**: Application layer ports, Django, DRF, Kafka, Redis, boto3, requests, celery, prometheus-client

### 2.4 Interface Layer (`interfaces/`)

**Location**: `backend/nexusasset/<context>/interfaces/`

**Contents**:
- `api/` — DRF ViewSets, Serializers, Routers (thin: authenticate → call use case → return response)
- `graphql/` — Strawberry schema (if used)
- `cli/` — Management commands (thin wrappers)
- `webhooks/` — Webhook receivers (verify signature → emit domain event)

**Allowed Imports**: Application layer use cases, DRF, Django, Strawberry

---

## 3. Module Structure (Monorepo Layout)

### 3.1 Backend Package Structure

```
backend/
└── nexusasset/
    ├── __init__.py
    ├── settings/                    # Django settings (base, dev, prod, test)
    │   ├── base.py
    │   ├── development.py
    │   ├── production.py
    │   └── testing.py
    ├── core/                        # nxa_core context
    │   ├── domain/
    │   │   ├── aggregates/          # FeatureFlag, AuditLogEntry, BackgroundJob
    │   │   ├── entities/
    │   │   ├── value_objects/
    │   │   ├── events/              # EVT-CR-###
    │   │   ├── services/            # DSV-CR-###
    │   │   ├── specifications/      # SPEC-CR-###
    │   │   ├── exceptions/
    │   │   └── ports/               # RepositoryPort, EventPublisherPort, CachePort
    │   ├── application/
    │   │   ├── use_cases/           # UC-CR-###
    │   │   ├── dto/
    │   │   ├── ports/               # FeatureFlagStore, AuditLogWriter, JobQueue
    │   │   ├── queries/
    │   │   └── mappers/
    │   ├── infrastructure/
    │   │   ├── repositories/        # Django ORM implementations
    │   │   ├── django/              # Models, migrations
    │   │   ├── messaging/           # Kafka outbox, consumers
    │   │   ├── cache/               # Redis adapter
    │   │   ├── metrics/             # Prometheus adapter
    │   │   └── celery/              # Background tasks
    │   └── interfaces/
    │       ├── api/                 # DRF ViewSets: FeatureFlagViewSet, AuditLogViewSet
    │       └── cli/
    │
    ├── accounts/                    # nxa_accounts context
    │   ├── domain/
    │   │   ├── aggregates/          # User, Role
    │   │   ├── entities/            # ApiKey, Session, MfaMethod
    │   │   ├── value_objects/       # UserId, Email, Permission, MfaMethod
    │   │   ├── events/              # EVT-AC-###
    │   │   ├── services/            # DSV-AC-###
    │   │   ├── specifications/      # SPEC-AC-###
    │   │   ├── exceptions/
    │   │   └── ports/               # UserRepository, RoleRepository, SessionRepository
    │   ├── application/
    │   │   ├── use_cases/           # UC-AC-### (AuthenticateUser, RegisterUser, AssignRole, etc.)
    │   │   ├── dto/
    │   │   ├── ports/               # PasswordHasher, TokenGenerator, MfaProvider, EmailSender
    │   │   ├── queries/
    │   │   └── mappers/
    │   ├── infrastructure/
    │   │   ├── repositories/        # DjangoUserRepository, DjangoRoleRepository
    │   │   ├── django/              # UserModel, RoleModel, ApiKeyModel
    │   │   ├── auth/                # Argon2idHasher, JwtTokenGenerator, TotpProvider, WebAuthnProvider
    │   │   ├── email/               # SendGridEmailSender
    │   │   └── celery/
    │   └── interfaces/
    │       ├── api/                 # AuthViewSet, UserViewSet, RoleViewSet, ApiKeyViewSet
    │       └── webhooks/
    │
    ├── tenants/                     # nxa_tenants context
    │   ├── domain/                  # Tenant, TenantUser, TenantSettings, QuotaLimits
    │   ├── application/             # UC-TN-### (ProvisionTenant, SuspendTenant, etc.)
    │   ├── infrastructure/          # DjangoTenantRepository, SchemaProvisioner, RlsPolicyInstaller
    │   └── interfaces/              # TenantViewSet, TenantUserViewSet
    │
    ├── assets/                      # nxa_assets context
    │   ├── domain/                  # Asset, CIRelationship, MaintenanceWindow, SoftwareInstallation
    │   ├── application/             # UC-AS-### (RegisterAsset, AuthorizeAsset, AssignAsset, etc.)
    │   ├── infrastructure/          # DjangoAssetRepository, DjangoCiRelationshipRepository
    │   └── interfaces/              # AssetViewSet, CiRelationshipViewSet, MaintenanceViewSet
    │
    ├── lansweeper/                  # nxa_lansweeper context
    │   ├── domain/                  # LansweeperSite, SyncJob, ReconciliationRule, FieldOwnershipRule
    │   ├── application/             # UC-LS-### (ConfigureSite, TriggerSync, ResolveConflict, etc.)
    │   ├── infrastructure/          # GraphQLClient, SyncOrchestrator, ReconciliationEngine
    │   └── interfaces/              # LansweeperSiteViewSet, SyncJobViewSet
    │
    ├── billing/                     # nxa_billing context
    │   ├── domain/                  # Subscription, Invoice, AllocationRule, Contract
    │   ├── application/             # UC-BL-### (CreateSubscription, GenerateInvoice, RunAllocation, etc.)
    │   ├── infrastructure/          # StripeClient, InvoicePdfGenerator, AllocationEngine
    │   └── interfaces/              # SubscriptionViewSet, InvoiceViewSet, ContractViewSet
    │
    ├── notifications/               # nxa_notifications context
    │   ├── domain/                  # NotificationChannel, NotificationRule, Notification
    │   ├── application/             # UC-NT-### (SendNotification, CreateRule, etc.)
    │   ├── infrastructure/          # EmailSender, SmsSender, SlackSender, WebhookSender, PushSender
    │   └── interfaces/              # NotificationChannelViewSet, NotificationRuleViewSet
    │
    └── audit/                       # nxa_audit context
        ├── domain/                  # Framework, Control, Assessment, Finding, EvidencePackage, Risk
        ├── application/             # UC-AU-### (RunAssessment, GenerateEvidence, ManageRisk, etc.)
        ├── infrastructure/          # OpaPolicyEngine, EvidencePackager, MerkleTreePublisher
        └── interfaces/              # FrameworkViewSet, AssessmentViewSet, RiskViewSet
```

### 3.2 Shared Kernel (`backend/nexusasset/shared/`)

```
shared/
├── kernel/
│   ├── domain/                      # Shared value objects, base classes
│   │   ├── base.py                  # AggregateRoot, Entity, ValueObject, DomainEvent
│   │   ├── value_objects.py         # UUIDv7, Money, Email, Url, Secret, IdempotencyKey
│   │   ├── events.py                # DomainEvent base, IntegrationEvent base
│   │   └── exceptions.py            # DomainException, InvariantViolation, NotFound
│   ├── application/
│   │   ├── use_case.py              # UseCase base class, UseCaseResult
│   │   ├── unit_of_work.py          # UnitOfWork protocol, Transactional decorator
│   │   ├── ports/                   # Shared ports: EventPublisher, Cache, FileStorage, HttpClient
│   │   └── dto.py                   # Base DTO classes, Pagination, Cursor
│   └── infrastructure/
│       ├── database/                # Django base model, tenant-aware manager
│       ├── messaging/               # Kafka producer base, outbox processor base
│       ├── cache/                   # Redis client base
│       ├── http/                    # HTTP client base with retry, circuit breaker
│       └── observability/           # Structured logging, tracing helpers
```

---

## 4. Use Case Catalog per Context

Each use case maps 1:1 to UC-### from DOC-02. Naming: `<Verb><Noun>UseCase` (e.g., `AuthorizeAssetUseCase`).

### 4.1 nxa_accounts Use Cases

| UC ID | Use Case Class | Input DTO | Output DTO | Primary Port | Secondary Ports |
|-------|----------------|-----------|------------|--------------|-----------------|
| UC-AC-001 | `AuthenticateUserUseCase` | `AuthenticateUserInput` | `AuthenticationResult` | `UserRepository` | `PasswordHasher`, `TokenGenerator`, `MfaProvider`, `SessionRepository` |
| UC-AC-002 | `SsoLoginUseCase` | `SsoLoginInput` | `AuthenticationResult` | `UserRepository` | `SamlValidator`, `OidcValidator`, `TokenGenerator` |
| UC-AC-003 | `UpdateProfileUseCase` | `UpdateProfileInput` | `UserProfileOutput` | `UserRepository` | — |
| UC-AC-004 | `CreateApiKeyUseCase` | `CreateApiKeyInput` | `ApiKeyOutput` | `UserRepository` | `ApiKeyGenerator`, `ApiKeyHasher` |
| UC-AC-005 | `AssignRoleUseCase` | `AssignRoleInput` | `RoleAssignmentOutput` | `UserRepository`, `RoleRepository` | `AuthorizationService` |
| UC-AC-006 | `ConfigureAbacPolicyUseCase` | `ConfigureAbacInput` | `AbacPolicyOutput` | `PolicyRepository` | `OpaClient` |
| UC-AC-007 | `StartImpersonationUseCase` | `StartImpersonationInput` | `ImpersonationSessionOutput` | `UserRepository` | `ApprovalWorkflow`, `AuditLogWriter` |
| UC-AC-008 | `RegisterMfaMethodUseCase` | `RegisterMfaInput` | `MfaMethodOutput` | `UserRepository` | `TotpProvider`, `WebAuthnProvider` |
| UC-AC-009 | `RevokeSessionUseCase` | `RevokeSessionInput` | `Void` | `SessionRepository` | — |
| UC-AC-010 | `ScimProvisionUseCase` | `ScimProvisionInput` | `ScimProvisionOutput` | `UserRepository`, `RoleRepository` | `ScimClient` |

### 4.2 nxa_tenants Use Cases

| UC ID | Use Case Class | Input DTO | Output DTO | Primary Port | Secondary Ports |
|-------|----------------|-----------|------------|--------------|-----------------|
| UC-TN-001 | `ProvisionTenantUseCase` | `ProvisionTenantInput` | `TenantOutput` | `TenantRepository` | `SchemaProvisioner`, `RlsPolicyInstaller`, `DefaultDataSeeder`, `EventPublisher` |
| UC-TN-002 | `UpdateTenantSettingsUseCase` | `UpdateTenantSettingsInput` | `TenantSettingsOutput` | `TenantRepository` | — |
| UC-TN-003 | `InviteTenantUserUseCase` | `InviteTenantUserInput` | `TenantUserOutput` | `TenantUserRepository` | `UserRepository`, `EmailSender`, `EventPublisher` |
| UC-TN-004 | `GetTenantUsageUseCase` | `GetTenantUsageInput` | `TenantUsageOutput` | `TenantRepository` | `MetricsQueryPort` |
| UC-TN-005 | `RequestQuotaIncreaseUseCase` | `RequestQuotaInput` | `QuotaRequestOutput` | `TenantRepository` | `ApprovalWorkflow`, `EventPublisher` |
| UC-TN-006 | `ExportTenantDataUseCase` | `ExportTenantDataInput` | `ExportJobOutput` | `TenantRepository` | `FileStorage`, `DataExporter`, `EventPublisher` |
| UC-TN-007 | `SuspendTenantUseCase` | `SuspendTenantInput` | `Void` | `TenantRepository` | `SessionRepository`, `EventPublisher` |
| UC-TN-008 | `OffboardTenantUseCase` | `OffboardTenantInput` | `OffboardingCertificate` | `TenantRepository` | `CryptoEraseService`, `SchemaDropper`, `EventPublisher` |
| UC-TN-009 | `ConfigureCustomDomainUseCase` | `ConfigureDomainInput` | `DomainVerificationOutput` | `TenantRepository` | `DnsVerifier`, `CertificateProvisioner`, `EventPublisher` |
| UC-TN-010 | `CreateChildTenantUseCase` | `CreateChildTenantInput` | `TenantOutput` | `TenantRepository` | `EventPublisher` |

### 4.3 nxa_assets Use Cases

| UC ID | Use Case Class | Input DTO | Output DTO | Primary Port | Secondary Ports |
|-------|----------------|-----------|------------|--------------|-----------------|
| UC-AS-001 | `RegisterAssetUseCase` | `RegisterAssetInput` | `AssetOutput` | `AssetRepository` | `AssetValidator`, `EventPublisher` |
| UC-AS-002 | `BulkImportAssetsUseCase` | `BulkImportInput` | `BulkImportResult` | `AssetRepository` | `CsvParser`, `AssetValidator`, `EventPublisher` |
| UC-AS-003 | `GetAssetDetailsUseCase` | `GetAssetInput` | `AssetDetailsOutput` | `AssetRepository` | `CiRelationshipRepository` |
| UC-AS-004 | `SearchAssetsUseCase` | `SearchAssetsInput` | `PaginatedAssetOutput` | `AssetRepository` | `AssetSearchPort` |
| UC-AS-005 | `AssignAssetUseCase` | `AssignAssetInput` | `AssetOutput` | `AssetRepository` | `UserRepository`, `EventPublisher` |
| UC-AS-006 | `ScheduleMaintenanceUseCase` | `ScheduleMaintenanceInput` | `MaintenanceWindowOutput` | `MaintenanceRepository` | `AssetRepository`, `UserRepository`, `EventPublisher` |
| UC-AS-007 | `RecordMaintenanceOutcomeUseCase` | `RecordMaintenanceInput` | `MaintenanceWindowOutput` | `MaintenanceRepository` | `AssetRepository`, `EventPublisher` |
| UC-AS-008 | `RelocateAssetUseCase` | `RelocateAssetInput` | `AssetOutput` | `AssetRepository` | `LocationRepository`, `EventPublisher` |
| UC-AS-009 | `RetireAssetUseCase` | `RetireAssetInput` | `AssetOutput` | `AssetRepository` | `SanitizationVerifier`, `EventPublisher` |
| UC-AS-010 | `ManageCiRelationshipUseCase` | `ManageCiRelationshipInput` | `CiRelationshipOutput` | `CiRelationshipRepository` | `AssetRepository`, `CycleDetector`, `EventPublisher` |
| UC-AS-011 | `CompareAssetVersionsUseCase` | `CompareAssetInput` | `AssetComparisonOutput` | `AssetRepository` | `AssetHistoryPort` |
| UC-AS-012 | `ManageCustomFieldsUseCase` | `ManageCustomFieldsInput` | `CustomFieldDefinitionOutput` | `CustomFieldRepository` | `TenantRepository`, `EventPublisher` |

### 4.4 nxa_lansweeper Use Cases

| UC ID | Use Case Class | Input DTO | Output DTO | Primary Port | Secondary Ports |
|-------|----------------|-----------|------------|--------------|-----------------|
| UC-LS-001 | `ConfigureLansweeperSiteUseCase` | `ConfigureSiteInput` | `LansweeperSiteOutput` | `SiteRepository` | `GraphQLClient`, `CredentialVault`, `EventPublisher` |
| UC-LS-002 | `TriggerSyncUseCase` | `TriggerSyncInput` | `SyncJobOutput` | `SyncJobRepository` | `SiteRepository`, `GraphQLClient`, `SyncOrchestrator`, `EventPublisher` |
| UC-LS-003 | `GetSyncStatusUseCase` | `GetSyncStatusInput` | `SyncStatusOutput` | `SyncJobRepository` | — |
| UC-LS-004 | `UpdateFieldOwnershipUseCase` | `UpdateFieldOwnershipInput` | `FieldOwnershipOutput` | `SiteRepository` | `ReconciliationRuleRepository`, `EventPublisher` |
| UC-LS-005 | `ResolveConflictUseCase` | `ResolveConflictInput` | `ConflictResolutionOutput` | `ConflictRepository` | `AssetRepository`, `ReconciliationEngine`, `EventPublisher` |
| UC-LS-006 | `RegisterWebhookUseCase` | `RegisterWebhookInput` | `WebhookRegistrationOutput` | `SiteRepository` | `WebhookVerifier`, `EventPublisher` |
| UC-LS-007 | `ProcessDeltaSyncUseCase` | `ProcessDeltaSyncInput` | `DeltaSyncOutput` | `SyncJobRepository` | `GraphQLClient`, `ReconciliationEngine`, `EventPublisher` |
| UC-LS-008 | `ReconcileSoftwareLicensesUseCase` | `ReconcileLicensesInput` | `LicenseReconciliationOutput` | `AssetRepository` | `LicenseEntitlementPort`, `EventPublisher` |
| UC-LS-009 | `CorrelateVulnerabilitiesUseCase` | `CorrelateVulnsInput` | `VulnerabilityReportOutput` | `AssetRepository` | `NvdClient`, `EventPublisher` |
| UC-LS-010 | `ManageScanCredentialsUseCase` | `ManageCredentialsInput` | `CredentialOutput` | `SiteRepository` | `CredentialVault`, `ConnectivityTester` |

### 4.5 nxa_billing Use Cases

| UC ID | Use Case Class | Input DTO | Output DTO | Primary Port | Secondary Ports |
|-------|----------------|-----------|------------|--------------|-----------------|
| UC-BL-001 | `ManagePlanUseCase` | `ManagePlanInput` | `PlanOutput` | `PlanRepository` | `EventPublisher` |
| UC-BL-002 | `PurchaseSubscriptionUseCase` | `PurchaseSubscriptionInput` | `SubscriptionOutput` | `SubscriptionRepository` | `StripeClient`, `TenantRepository`, `EventPublisher` |
| UC-BL-003 | `GetInvoicesUseCase` | `GetInvoicesInput` | `PaginatedInvoiceOutput` | `InvoiceRepository` | — |
| UC-BL-004 | `ConfigureAllocationRuleUseCase` | `ConfigureAllocationInput` | `AllocationRuleOutput` | `AllocationRuleRepository` | `Validator`, `EventPublisher` |
| UC-BL-005 | `GenerateChargebackReportUseCase` | `GenerateReportInput` | `ChargebackReportOutput` | `AllocationRuleRepository` | `AssetRepository`, `InvoiceRepository`, `ReportGenerator` |
| UC-BL-006 | `ManageContractUseCase` | `ManageContractInput` | `ContractOutput` | `ContractRepository` | `DocumentStorage`, `EventPublisher` |
| UC-BL-007 | `RunLicenseTrueUpUseCase` | `RunTrueUpInput` | `TrueUpResultOutput` | `AssetRepository` | `LicenseEntitlementPort`, `ContractRepository`, `EventPublisher` |
| UC-BL-008 | `ConfigureMultiCurrencyUseCase` | `ConfigureCurrencyInput` | `CurrencyConfigOutput` | `TenantRepository` | `FxRateProvider`, `TaxEngine` |
| UC-BL-009 | `ManagePaymentMethodUseCase` | `ManagePaymentMethodInput` | `PaymentMethodOutput` | `SubscriptionRepository` | `StripeClient` |
| UC-BL-010 | `GetFinancialDashboardUseCase` | `GetDashboardInput` | `FinancialDashboardOutput` | `InvoiceRepository` | `SubscriptionRepository`, `MetricsQueryPort` |

### 4.6 nxa_notifications Use Cases

| UC ID | Use Case Class | Input DTO | Output DTO | Primary Port | Secondary Ports |
|-------|----------------|-----------|------------|--------------|-----------------|
| UC-NT-001 | `ConfigureChannelUseCase` | `ConfigureChannelInput` | `ChannelOutput` | `ChannelRepository` | `ChannelValidator`, `EventPublisher` |
| UC-NT-002 | `CreateNotificationRuleUseCase` | `CreateRuleInput` | `RuleOutput` | `RuleRepository` | `ChannelRepository`, `EventPublisher` |
| UC-NT-003 | `UpdateUserPreferencesUseCase` | `UpdatePreferencesInput` | `PreferencesOutput` | `UserPreferenceRepository` | — |
| UC-NT-004 | `GetNotificationHistoryUseCase` | `GetHistoryInput` | `PaginatedNotificationOutput` | `NotificationRepository` | — |
| UC-NT-005 | `BroadcastAnnouncementUseCase` | `BroadcastInput` | `BroadcastResultOutput` | `NotificationRepository` | `ChannelRepository`, `DeliveryService`, `EventPublisher` |
| UC-NT-006 | `ProcessNotificationQueueUseCase` | `ProcessQueueInput` | `Void` | `NotificationRepository` | `DeliveryService`, `EventPublisher` |

### 4.7 nxa_audit Use Cases

| UC ID | Use Case Class | Input DTO | Output DTO | Primary Port | Secondary Ports |
|-------|----------------|-----------|------------|--------------|-----------------|
| UC-AU-001 | `ManageFrameworkUseCase` | `ManageFrameworkInput` | `FrameworkOutput` | `FrameworkRepository` | `EventPublisher` |
| UC-AU-002 | `MapControlToPolicyUseCase` | `MapControlInput` | `ControlMappingOutput` | `ControlRepository` | `OpaClient`, `EventPublisher` |
| UC-AU-003 | `RunAssessmentUseCase` | `RunAssessmentInput` | `AssessmentOutput` | `AssessmentRepository` | `FrameworkRepository`, `OpaPolicyEngine`, `EvidenceCollector`, `EventPublisher` |
| UC-AU-004 | `GenerateEvidencePackageUseCase` | `GenerateEvidenceInput` | `EvidencePackageOutput` | `EvidencePackageRepository` | `MerkleTreePublisher`, `SigningService`, `FileStorage`, `EventPublisher` |
| UC-AU-005 | `ManageRiskUseCase` | `ManageRiskInput` | `RiskOutput` | `RiskRepository` | `EventPublisher` |
| UC-AU-006 | `ManageIncidentUseCase` | `ManageIncidentInput` | `IncidentOutput` | `IncidentRepository` | `EvidencePreserver`, `NotificationService`, `EventPublisher` |
| UC-AU-007 | `ClassifyDataUseCase` | `ClassifyDataInput` | `ClassificationOutput` | `DataAssetRepository` | `DlpEngine`, `EventPublisher` |
| UC-AU-008 | `AssessVendorRiskUseCase` | `AssessVendorInput` | `VendorAssessmentOutput` | `VendorRepository` | `QuestionnaireEngine`, `SecurityRatingClient`, `EventPublisher` |
| UC-AU-009 | `ManageOpaPolicyUseCase` | `ManagePolicyInput` | `PolicyOutput` | `PolicyRepository` | `OpaClient`, `GitRepository`, `EventPublisher` |
| UC-AU-010 | `GetComplianceDashboardUseCase` | `GetDashboardInput` | `ComplianceDashboardOutput` | `AssessmentRepository` | `FrameworkRepository`, `MetricsQueryPort` |

### 4.8 nxa_core Use Cases

| UC ID | Use Case Class | Input DTO | Output DTO | Primary Port | Secondary Ports |
|-------|----------------|-----------|------------|--------------|-----------------|
| UC-CR-001 | `ConfigureApiGatewayUseCase` | `ConfigureGatewayInput` | `GatewayConfigOutput` | `GatewayConfigRepository` | `EventPublisher` |
| UC-CR-002 | `ManageDeveloperPortalUseCase` | `ManagePortalInput` | `PortalConfigOutput` | `ApiSpecRepository` | `SdkGenerator`, `EventPublisher` |
| UC-CR-003 | `ManageFeatureFlagUseCase` | `ManageFlagInput` | `FeatureFlagOutput` | `FeatureFlagRepository` | `EventPublisher` |
| UC-CR-004 | `UpdateStatusPageUseCase` | `UpdateStatusInput` | `StatusPageOutput` | `ComponentStatusRepository` | `EventPublisher` |
| UC-CR-005 | `QueryAuditLogUseCase` | `QueryAuditLogInput` | `PaginatedAuditLogOutput` | `AuditLogRepository` | — |
| UC-CR-006 | `ManageBackgroundJobUseCase` | `ManageJobInput` | `JobOutput` | `JobRepository` | `JobQueue`, `EventPublisher` |

---

## 5. Port Contracts (Interfaces)

Ports are **defined in the Application layer** (or Domain layer for repositories) and **implemented in Infrastructure**.

### 5.1 Repository Ports (Domain Layer → Infrastructure)

```python
# backend/nexusasset/shared/kernel/domain/ports.py
from abc import ABC, abstractmethod
from typing import Generic, TypeVar, Optional, List
from nexusasset.shared.kernel.domain.base import AggregateRoot, Entity

T = TypeVar('T', bound=AggregateRoot)
E = TypeVar('E', bound=Entity)

class RepositoryPort(ABC, Generic[T]):
    """Base repository port for aggregate roots."""
    
    @abstractmethod
    async def add(self, aggregate: T) -> None:
        """Persist new aggregate."""
    
    @abstractmethod
    async def get(self, id: T.Id) -> Optional[T]:
        """Retrieve aggregate by ID."""
    
    @abstractmethod
    async def update(self, aggregate: T) -> None:
        """Update existing aggregate."""
    
    @abstractmethod
    async def delete(self, id: T.Id) -> None:
        """Delete aggregate (soft/hard per implementation)."""
    
    @abstractmethod
    async def exists(self, id: T.Id) -> bool:
        """Check existence."""

class QueryRepositoryPort(ABC, Generic[T]):
    """Read-optimized repository for queries (CQRS)."""
    
    @abstractmethod
    async def find_by_criteria(self, criteria: 'QueryCriteria') -> List[T]:
        pass
    
    @abstractmethod
    async def count_by_criteria(self, criteria: 'QueryCriteria') -> int:
        pass

# Context-specific repository ports defined in each domain/ports/
# e.g., backend/nexusasset/accounts/domain/ports/user_repository.py
class UserRepositoryPort(RepositoryPort[User]):
    @abstractmethod
    async def get_by_email(self, email: Email, tenant_id: TenantId) -> Optional[User]:
        pass
    
    @abstractmethod
    async def get_by_api_key_hash(self, key_hash: ApiKeyHash) -> Optional[User]:
        pass
    
    @abstractmethod
    async def list_by_tenant(self, tenant_id: TenantId, pagination: Pagination) -> List[User]:
        pass
```

### 5.2 Secondary Ports (Application Layer → Infrastructure)

```python
# backend/nexusasset/shared/kernel/application/ports/event_publisher.py
from abc import ABC, abstractmethod
from nexusasset.shared.kernel.domain.events import IntegrationEvent

class EventPublisherPort(ABC):
    """Publish integration events to message broker."""
    
    @abstractmethod
    async def publish(self, event: IntegrationEvent) -> None:
        """Publish single event."""
    
    @abstractmethod
    async def publish_batch(self, events: List[IntegrationEvent]) -> None:
        """Publish multiple events atomically (transactional outbox)."""

# backend/nexusasset/shared/kernel/application/ports/cache.py
class CachePort(ABC):
    @abstractmethod
    async def get(self, key: str) -> Optional[bytes]:
        pass
    
    @abstractmethod
    async def set(self, key: str, value: bytes, ttl_seconds: int) -> None:
        pass
    
    @abstractmethod
    async def delete(self, key: str) -> None:
        pass
    
    @abstractmethod
    async def exists(self, key: str) -> bool:
        pass

# backend/nexusasset/accounts/application/ports/auth_ports.py
class PasswordHasherPort(ABC):
    @abstractmethod
    def hash(self, password: str) -> PasswordHash:
        pass
    
    @abstractmethod
    def verify(self, password: str, hash: PasswordHash) -> bool:
        pass

class TokenGeneratorPort(ABC):
    @abstractmethod
    def generate_access_token(self, user: User, tenant: Tenant) -> AccessToken:
        pass
    
    @abstractmethod
    def generate_refresh_token(self, user: User) -> RefreshToken:
        pass
    
    @abstractmethod
    def decode_access_token(self, token: str) -> TokenClaims:
        pass

class MfaProviderPort(ABC):
    @abstractmethod
    def generate_totp_secret(self) -> TotpSecret:
        pass
    
    @abstractmethod
    def verify_totp(self, secret: TotpSecret, code: str) -> bool:
        pass
    
    @abstractmethod
    def generate_webauthn_options(self, user: User) -> WebAuthnOptions:
        pass
    
    @abstractmethod
    def verify_webauthn(self, credential: WebAuthnCredential) -> bool:
        pass

# backend/nexusasset/lansweeper/application/ports/sync_ports.py
class GraphQLClientPort(ABC):
    @abstractmethod
    async def query_assets(self, site: LansweeperSite, cursor: Optional[str], filter: AssetFilter) -> GraphQLResponse:
        pass
    
    @abstractmethod
    async def query_asset_by_id(self, site: LansweeperSite, external_id: str) -> Optional[GraphQLAsset]:
        pass

class ReconciliationEnginePort(ABC):
    @abstractmethod
    async def reconcile(self, discovered_asset: DiscoveredAsset, existing_candidates: List[Asset]) -> ReconciliationResult:
        pass
```

---

## 6. Adapter Implementations

Adapters live in `infrastructure/` and implement ports. Key patterns:

### 6.1 Django ORM Repository Adapter

```python
# backend/nexusasset/accounts/infrastructure/repositories/django_user_repository.py
from nexusasset.accounts.domain.ports.user_repository import UserRepositoryPort
from nexusasset.accounts.domain.aggregates.user import User, UserId
from nexusasset.accounts.infrastructure.django.models import UserModel
from nexusasset.shared.kernel.infrastructure.database import TenantAwareManager

class DjangoUserRepository(UserRepositoryPort):
    def __init__(self, tenant_id: TenantId):
        self.tenant_id = tenant_id
        self._manager = UserModel.objects.for_tenant(tenant_id)
    
    async def add(self, user: User) -> None:
        model = UserModel.from_domain(user)
        await sync_to_async(model.save)()
    
    async def get(self, user_id: UserId) -> Optional[User]:
        try:
            model = await sync_to_async(self._manager.get)(id=user_id.value)
            return model.to_domain()
        except UserModel.DoesNotExist:
            return None
    
    async def get_by_email(self, email: Email, tenant_id: TenantId) -> Optional[User]:
        try:
            model = await sync_to_async(self._manager.get)(email=email.value, tenant_id=tenant_id.value)
            return model.to_domain()
        except UserModel.DoesNotExist:
            return None
    
    async def update(self, user: User) -> None:
        model = await sync_to_async(self._manager.get)(id=user.id.value)
        model.update_from_domain(user)
        await sync_to_async(model.save)()
    
    # ... other methods
```

### 6.2 Kafka Event Publisher (Transactional Outbox)

```python
# backend/nexusasset/shared/kernel/infrastructure/messaging/kafka_event_publisher.py
from nexusasset.shared.kernel.application.ports.event_publisher import EventPublisherPort
from nexusasset.shared.kernel.domain.events import IntegrationEvent
from nexusasset.shared.kernel.infrastructure.database import OutboxEntry

class KafkaEventPublisher(EventPublisherPort):
    def __init__(self, producer: AIOKafkaProducer, outbox_repo: OutboxRepository):
        self.producer = producer
        self.outbox_repo = outbox_repo
    
    async def publish(self, event: IntegrationEvent) -> None:
        # Write to outbox table within same DB transaction as use case
        entry = OutboxEntry(
            event_type=type(event).__name__,
            payload=event.model_dump_json(),
            correlation_id=event.correlation_id,
            causation_id=event.causation_id,
            tenant_id=event.tenant_id,
        )
        await self.outbox_repo.add(entry)
        # Actual Kafka send happens in separate outbox processor
    
    async def publish_batch(self, events: List[IntegrationEvent]) -> None:
        entries = [OutboxEntry(...) for event in events]
        await self.outbox_repo.add_batch(entries)

# Outbox Processor (runs as separate Celery beat task)
class OutboxProcessor:
    async def process_batch(self, batch_size: int = 100) -> int:
        entries = await self.outbox_repo.get_unpublished(batch_size)
        for entry in entries:
            await self.producer.send_and_wait(
                topic=f"nxa.{entry.tenant_id}.{entry.event_type.lower()}",
                key=entry.correlation_id.encode(),
                value=entry.payload.encode(),
                headers=[("event_type", entry.event_type.encode())]
            )
            await self.outbox_repo.mark_published(entry.id)
        return len(entries)
```

### 6.3 HTTP Client Adapters (Resilient)

```python
# backend/nexusasset/lansweeper/infrastructure/http/graphql_client.py
from nexusasset.lansweeper.application.ports.sync_ports import GraphQLClientPort
from nexusasset.shared.kernel.infrastructure.http import ResilientHttpClient

class LansweeperGraphQLClient(GraphQLClientPort):
    def __init__(self, http_client: ResilientHttpClient, vault: CredentialVault):
        self.http = http_client
        self.vault = vault
    
    async def query_assets(self, site: LansweeperSite, cursor: Optional[str], filter: AssetFilter) -> GraphQLResponse:
        credentials = await self.vault.get(site.auth_config.credential_ref)
        headers = {"Authorization": f"Bearer {credentials.token}"}
        
        query = self._build_assets_query(cursor, filter)
        
        response = await self.http.post(
            url=site.graphql_endpoint,
            json={"query": query},
            headers=headers,
            timeout=30,
            retries=3,
            circuit_breaker=True,
        )
        return GraphQLResponse.from_json(response.json())
```

---

## 7. Cross-Cutting Concerns

### 7.1 Tenant Context Resolution

```python
# backend/nexusasset/shared/kernel/infrastructure/database/tenant_context.py
class TenantContextMiddleware:
    """Django middleware to resolve tenant from request."""
    
    def __call__(self, request):
        # Priority: subdomain → header → JWT claim
        tenant_slug = self._extract_tenant_slug(request)
        if tenant_slug:
            tenant = TenantService.get_by_slug(tenant_slug)
            request.tenant = tenant
            set_tenant_context(tenant.id)  # Thread-local for RLS
        return self.get_response(request)

def set_tenant_context(tenant_id: TenantId) -> None:
    """Sets PostgreSQL session variable for RLS."""
    from django.db import connection
    with connection.cursor() as cursor:
        cursor.execute("SET LOCAL app.tenant_id = %s", [str(tenant_id.value)])
```

### 7.2 Transaction Management (Unit of Work)

```python
# backend/nexusasset/shared/kernel/application/unit_of_work.py
from contextlib import asynccontextmanager
from abc import ABC, abstractmethod

class UnitOfWork(ABC):
    """Unit of Work protocol for transaction management."""
    
    @abstractmethod
    @asynccontextmanager
    async def transaction(self):
        """Yield transaction context; commit on success, rollback on exception."""
    
    @abstractmethod
    async def commit(self) -> None:
        pass
    
    @abstractmethod
    async def rollback(self) -> None:
        pass

# Django implementation
class DjangoUnitOfWork(UnitOfWork):
    def __init__(self):
        self._transaction = None
    
    @asynccontextmanager
    async def transaction(self):
        async with transaction.atomic():
            yield self
    
    async def commit(self) -> None:
        pass  # Handled by atomic()
    
    async def rollback(self) -> None:
        pass  # Handled by atomic()

# Decorator for use cases
def transactional(use_case_method):
    @wraps(use_case_method)
    async def wrapper(self, *args, **kwargs):
        async with self.uow.transaction():
            return await use_case_method(self, *args, **kwargs)
    return wrapper

# Usage in use case
class AuthorizeAssetUseCase:
    def __init__(self, uow: UnitOfWork, asset_repo: AssetRepositoryPort, event_pub: EventPublisherPort):
        self.uow = uow
        self.asset_repo = asset_repo
        self.event_pub = event_pub
    
    @transactional
    async def execute(self, input: AuthorizeAssetInput) -> AssetOutput:
        asset = await self.asset_repo.get(input.asset_id)
        asset.authorize(input.actor_id, input.reason)
        await self.asset_repo.update(asset)
        await self.event_pub.publish(AssetAuthorizedEvent(...))
        return AssetOutput.from_domain(asset)
```

### 7.3 Authorization in Use Cases

```python
# backend/nexusasset/shared/kernel/application/authorization.py
class AuthorizationService:
    def __init__(self, policy_engine: OpaClient):
        self.policy_engine = policy_engine
    
    async def authorize(self, actor: UserId, action: str, resource: ResourceInfo, tenant: TenantId) -> bool:
        input = {
            "actor": {"id": str(actor), "roles": await self._get_roles(actor, tenant)},
            "action": action,
            "resource": {"type": resource.type, "id": str(resource.id), "tenant_id": str(tenant)},
            "tenant": {"id": str(tenant)},
        }
        result = await self.policy_engine.evaluate("authorization", input)
        return result.allow

# In use case
class AssignAssetUseCase:
    @transactional
    async def execute(self, input: AssignAssetInput) -> AssetOutput:
        # Check permission
        allowed = await self.auth_service.authorize(
            actor=input.actor_id,
            action="assets:assign",
            resource=ResourceInfo(type="Asset", id=input.asset_id),
            tenant=input.tenant_id
        )
        if not allowed:
            raise AuthorizationDeniedError("Insufficient permissions to assign asset")
        # ... proceed
```

---

## 8. Dependency Injection & Composition Root

### 8.1 Container Configuration (`backend/nexusasset/composition_root.py`)

```python
# backend/nexusasset/composition_root.py
from dependency_injector import containers, providers
from nexusasset.shared.kernel.infrastructure.database import DjangoUnitOfWork
from nexusasset.shared.kernel.infrastructure.messaging import KafkaEventPublisher
from nexusasset.shared.kernel.infrastructure.cache import RedisCache
from nexusasset.accounts.infrastructure.repositories import DjangoUserRepository, DjangoRoleRepository
from nexusasset.accounts.infrastructure.auth import Argon2idHasher, JwtTokenGenerator, PyTotpProvider, WebAuthnProvider
from nexusasset.accounts.application.use_cases import (
    AuthenticateUserUseCase, SsoLoginUseCase, AssignRoleUseCase, ...
)

class CoreContainer(containers.DeclarativeContainer):
    config = providers.Configuration()
    
    # Infrastructure
    uow = providers.Factory(DjangoUnitOfWork)
    event_publisher = providers.Singleton(KafkaEventPublisher)
    cache = providers.Singleton(RedisCache)
    
class AccountsContainer(containers.DeclarativeContainer):
    config = providers.Configuration()
    
    # Repositories
    user_repo = providers.Factory(DjangoUserRepository, tenant_id=config.tenant_id)
    role_repo = providers.Factory(DjangoRoleRepository, tenant_id=config.tenant_id)
    session_repo = providers.Factory(DjangoSessionRepository, tenant_id=config.tenant_id)
    
    # Auth adapters
    password_hasher = providers.Singleton(Argon2idHasher)
    token_generator = providers.Singleton(JwtTokenGenerator, config=config.jwt)
    totp_provider = providers.Singleton(PyTotpProvider)
    webauthn_provider = providers.Singleton(WebAuthnProvider)
    
    # Use Cases
    authenticate_user = providers.Factory(
        AuthenticateUserUseCase,
        uow=CoreContainer.uow,
        user_repo=user_repo,
        password_hasher=password_hasher,
        token_generator=token_generator,
        totp_provider=totp_provider,
        webauthn_provider=webauthn_provider,
        session_repo=session_repo,
        event_publisher=CoreContainer.event_publisher,
    )
    
    assign_role = providers.Factory(
        AssignRoleUseCase,
        uow=CoreContainer.uow,
        user_repo=user_repo,
        role_repo=role_repo,
        event_publisher=CoreContainer.event_publisher,
    )

# Similar containers for each context...
```

### 8.2 Django Integration (`backend/nexusasset/di.py`)

```python
# backend/nexusasset/di.py
from dependency_injector.wiring import inject, Provide
from .composition_root import Container

container = Container()
container.config.from_yaml("config.yml")
container.wire(modules=[__name__])

@inject
def get_use_case(use_case_class, 
                 uow=Provide[Container.core.uow],
                 event_publisher=Provide[Container.core.event_publisher],
                 **repositories):
    return use_case_class(uow=uow, event_publisher=event_publisher, **repositories)
```

---

## 9. Transaction Management & Unit of Work

### 9.1 Transaction Boundaries

| Boundary | Scope | Implementation |
|----------|-------|----------------|
| **Use Case** | Single business operation (UC-###) | `@transactional` decorator on `execute()` |
| **Saga** | Multi-step distributed operation (e.g., tenant provisioning) | Explicit compensation actions; orchestrator use case |
| **Event Handler** | Processing single integration event | Separate transaction per handler; idempotency via event ID |
| **Background Job** | Celery task execution | New transaction per task; retry with new transaction |

### 9.2 Concurrency Control

```python
# Optimistic locking via version field on aggregates
class Asset(AggregateRoot[AssetId]):
    version: int  # Incremented on every change
    
    def _increment_version(self) -> None:
        self.version += 1

# Repository checks version on update
async def update(self, asset: Asset) -> None:
    updated = await AssetModel.objects.filter(
        id=asset.id.value, 
        version=asset.version - 1  # Expected previous version
    ).aupdate(**asset.to_dict(), version=asset.version)
    if updated == 0:
        raise OptimisticLockError("Asset was modified concurrently")
```

---

## 10. Event Publishing & Outbox Pattern

### 10.1 Domain Events → Integration Events

```python
# In aggregate method
def authorize(self, actor: UserId, reason: str) -> None:
    self._validate_authorization_transition()
    self.state = AssetState.AUTHORIZED
    self.state_changed_at = DateTime.now()
    self.state_changed_by = actor
    self.state_reason = reason
    self._increment_version()
    
    # Raise domain event (collected by use case)
    self.add_domain_event(AssetAuthorizedEvent(
        asset_id=self.id,
        tenant_id=self.tenant_id,
        authorized_by=actor,
        reason=reason,
        timestamp=DateTime.now(),
    ))

# In use case (after successful transaction)
@transactional
async def execute(self, input: AuthorizeAssetInput) -> AssetOutput:
    asset = await self.asset_repo.get(input.asset_id)
    asset.authorize(input.actor_id, input.reason)
    await self.asset_repo.update(asset)
    
    # Publish domain events as integration events
    for event in asset.domain_events:
        integration_event = self._to_integration_event(event)
        await self.event_pub.publish(integration_event)
    
    asset.clear_domain_events()
    return AssetOutput.from_domain(asset)
```

### 10.2 Outbox Table Schema

```sql
-- backend/nexusasset/shared/kernel/infrastructure/database/migrations/0001_outbox.py
CREATE TABLE outbox (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_type VARCHAR(200) NOT NULL,
    payload JSONB NOT NULL,
    correlation_id UUID NOT NULL,
    causation_id UUID,
    tenant_id UUID NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    published_at TIMESTAMPTZ,
    attempts INT NOT NULL DEFAULT 0,
    last_error TEXT
);

CREATE INDEX idx_outbox_unpublished ON outbox (published_at) WHERE published_at IS NULL;
CREATE INDEX idx_outbox_tenant ON outbox (tenant_id, created_at);
```

---

## 11. Error Handling & Result Types

### 11.1 Result Type (No Exceptions for Control Flow)

```python
# backend/nexusasset/shared/kernel/application/result.py
from dataclasses import dataclass
from typing import Generic, TypeVar, Optional

T = TypeVar('T')
E = TypeVar('E', bound='ApplicationError')

@dataclass(frozen=True)
class Result(Generic[T, E]):
    """Result type for use case returns — no exceptions for expected failures."""
    value: Optional[T] = None
    error: Optional[E] = None
    
    @property
    def is_success(self) -> bool:
        return self.error is None
    
    @property
    def is_failure(self) -> bool:
        return self.error is not None
    
    @staticmethod
    def ok(value: T) -> 'Result[T, E]':
        return Result(value=value)
    
    @staticmethod
    def fail(error: E) -> 'Result[T, E]':
        return Result(error=error)

class ApplicationError(Exception):
    """Base for expected application errors (not bugs)."""
    code: str
    message: str
    details: dict

class NotFoundError(ApplicationError):
    code = "NOT_FOUND"

class AuthorizationDeniedError(ApplicationError):
    code = "AUTHORIZATION_DENIED"

class InvariantViolationError(ApplicationError):
    code = "INVARIANT_VIOLATION"

class ConcurrencyError(ApplicationError):
    code = "CONCURRENCY_CONFLICT"

# Usage in use case
async def execute(self, input: AssignAssetInput) -> Result[AssetOutput, ApplicationError]:
    asset = await self.asset_repo.get(input.asset_id)
    if asset is None:
        return Result.fail(NotFoundError("Asset not found", details={"asset_id": str(input.asset_id)}))
    
    allowed = await self.auth_service.authorize(...)
    if not allowed:
        return Result.fail(AuthorizationDeniedError("Cannot assign asset"))
    
    try:
        asset.assign(input.owner_id, input.custodian_id, input.actor_id)
    except InvariantViolation as e:
        return Result.fail(InvariantViolationError(str(e)))
    
    await self.asset_repo.update(asset)
    await self.event_pub.publish(...)
    return Result.ok(AssetOutput.from_domain(asset))
```

---

## 12. Architecture Decision Records (ADRs)

| ADR ID | Title | Status | Context | Decision | Consequences |
|--------|-------|--------|---------|----------|--------------|
| **ADR-001** | Clean Architecture with DDD | Accepted | Need maintainable, testable architecture | Enforce Dependency Rule; Domain pure Python; Use Cases orchestrate | More boilerplate; slower initial dev; long-term velocity |
| **ADR-002** | Transactional Outbox for Events | Accepted | Reliable event publishing without dual-write | Write events to outbox table in same DB transaction; separate processor publishes to Kafka | Added latency (~100ms); outbox processor operational complexity |
| **ADR-003** | Django ORM Only in Infrastructure | Accepted | Domain purity requirement | Repository implementations use Django ORM; Domain uses abstract ports | Repository tests need testcontainers; migration coupling |
| **ADR-004** | Per-Tenant RLS for Isolation | Accepted | Multi-tenancy security requirement | PostgreSQL RLS policies on all tenant tables; `SET LOCAL app.tenant_id` | Query complexity; connection pooling considerations |
| **ADR-005** | Result Type Over Exceptions | Accepted | Control flow clarity | Use `Result<T, E>` for expected failures; exceptions only for bugs | More verbose; explicit error handling |
| **ADR-006** | CQRS for Read Models | Accepted | Complex queries, performance | Separate query repositories with optimized read models | Eventual consistency; dual write complexity |
| **ADR-007** | Feature Flags via nxa_core | Accepted | Safe rollouts, kill switches | Boolean/multivariate flags with targeting rules; evaluated at edge | Additional latency; flag explosion risk |
| **ADR-008** | Idempotency Keys for All Mutations | Accepted | Exactly-once semantics | Every mutating command requires idempotency key; stored in Redis/DB | Client complexity; storage growth |
| **ADR-009** | Policy-as-Code (OPA) for AuthZ | Accepted | Fine-grained, auditable authorization | OPA/Rego policies; evaluated in AuthorizationService; decisions logged | Policy learning curve; performance (mitigated by caching) |
| **ADR-010** | Background Jobs via Celery + Outbox | Accepted | Reliable async processing | Celery for scheduling; use cases called from tasks; events via outbox | Celery operational complexity; task visibility |

---

## 13. Diagrams

| Diagram ID | Title | Type | Location | Description |
|------------|-------|------|----------|-------------|
| **DIA-046** | Clean Architecture Layers | Layered | Section 1.1 | Four layers with dependency direction |
| **DIA-047** | Module Structure | Package Diagram | Section 3.1 | Backend package tree |
| **DIA-048** | Use Case Flow (AuthorizeAsset) | Sequence | Section 4.3 | Controller → Use Case → Domain → Repo → Event |
| **DIA-049** | Port/Adapter Pattern | Component | Section 5 | Application defines ports; Infrastructure implements |
| **DIA-050** | Django Repository Adapter | Class Diagram | Section 6.1 | Domain Port → Django Implementation |
| **DIA-051** | Transactional Outbox | Sequence | Section 10 | Use Case → DB Transaction → Outbox → Kafka |
| **DIA-052** | Tenant Context Resolution | Sequence | Section 7.1 | Request → Middleware → RLS → Repository |
| **DIA-053** | Dependency Injection Graph | Component | Section 8 | Container → Use Cases → Ports → Adapters |
| **DIA-054** | Authorization Flow | Sequence | Section 7.3 | Use Case → Auth Service → OPA → Decision |
| **DIA-055** | Result Type Flow | Flowchart | Section 11 | Success/Failure paths in use case |
| **DIA-056** | Saga: Tenant Provisioning | Sequence | Section 9.1 | Orchestrator → Steps → Compensations |

---

## 14. Tables

| Table ID | Title | Location | Row Count |
|----------|-------|----------|-----------|
| **TBL-113** | Architectural Principles | Section 1.2 | 8 |
| **TBL-114** | Layer Definitions | Section 2 | 4 |
| **TBL-115** | nxa_accounts Use Cases | Section 4.1 | 10 |
| **TBL-116** | nxa_tenants Use Cases | Section 4.2 | 10 |
| **TBL-117** | nxa_assets Use Cases | Section 4.3 | 12 |
| **TBL-118** | nxa_lansweeper Use Cases | Section 4.4 | 10 |
| **TBL-119** | nxa_billing Use Cases | Section 4.5 | 10 |
| **TBL-120** | nxa_notifications Use Cases | Section 4.6 | 6 |
| **TBL-121** | nxa_audit Use Cases | Section 4.7 | 10 |
| **TBL-122** | nxa_core Use Cases | Section 4.8 | 6 |
| **TBL-123** | Repository Port Methods | Section 5.1 | 5 |
| **TBL-124** | Secondary Port Contracts | Section 5.2 | 8 |
| **TBL-125** | Adapter Implementations | Section 6 | 6 |
| **TBL-126** | Cross-Cutting Concerns | Section 7 | 5 |
| **TBL-127** | Transaction Boundaries | Section 9.1 | 4 |
| **TBL-128** | ADR Register | Section 12 | 10 |
| **TBL-129** | Diagram Index | Section 13 | 11 |
| **TBL-130** | Table Index | Section 14 | 18 |

---

---

**End of DOC-05 — NexusAsset Cloud Application Architecture**

*Document Control: Version 1.0.0 | Classification: Internal — Confidential | Next Review: 2026-12-09*

*For questions or updates: docs@nexusasset.cloud | Architecture Review Board: arch-review@nexusasset.cloud*