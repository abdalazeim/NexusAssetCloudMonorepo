╔═════════════════════════════════════════════════════════════════════════════════════════════════╗
║ Document ID: DOC-04                                                                            ║
║ Title: Domain Model — Bounded Contexts, Aggregates, Entities, Value Objects, Events            ║
║ Version: 1.0.0                                                                                 ║
║ Status: ✓ Approved                                                                             ║
║ Classification: Internal — Confidential                                                        ║
║ Authors: NexusAsset Cloud Architecture Team                                                    ║
║ Reviewers: Domain Experts, Engineering Leads, Security, QA                                     ║
║ Approved By: CTO / Chief Architect                                                             ║
║ Date: 2026-11-09                                                                               ║
║ Supersedes: —                                                                                  ║
║ Related: DOC-00, DOC-01, DOC-02, DOC-03, DOC-05, DOC-06, DOC-07, DOC-08, DOC-13              ║
║ Traceability: ENT/CTX-### ← CAP-###/REQ-FN-###, VO/CTX-###, AGG/CTX-###, EVT/CTX-###, IEV/CTX-### ║
╚═════════════════════════════════════════════════════════════════════════════════════════════════╝

# ═══════════════════════════════════════════════════════════════
# NexusAsset Cloud — Domain Model (DOC-04)
# ═══════════════════════════════════════════════════════════════

> **Executive Summary** — This document defines the canonical domain model for NexusAsset Cloud using Domain-Driven Design (DDD) principles. It specifies 8 bounded contexts, their aggregates, entities, value objects, domain events, and integration events. The model enforces the **Dependency Rule**: domain layer has zero dependencies on Django, ORM, or infrastructure. All identifiers follow the stable scheme: `ENT/CTX-###` (entities), `VO/CTX-###` (value objects), `AGG/CTX-###` (aggregates), `EVT/CTX-###` (domain events), `IEV/CTX-###` (integration events). This model is the single source of truth for database schema (DOC-06), API contracts (DOC-07), and application services (DOC-05).

---

## Table of Contents

1. [Bounded Context Map](#1-bounded-context-map)
2. [nxa_accounts — Identity & Access Context](#2-nxa_accounts--identity--access-context)
3. [nxa_tenants — Tenant Management Context](#3-nxa_tenants--tenant-management-context)
4. [nxa_assets — Asset Core Context](#4-nxa_assets--asset-core-context)
5. [nxa_lansweeper — Lansweeper Integration Context](#5-nxa_lansweeper--lansweeper-integration-context)
6. [nxa_billing — Billing & Contracts Context](#6-nxa_billing--billing--contracts-context)
7. [nxa_notifications — Notifications Context](#7-nxa_notifications--notifications-context)
8. [nxa_audit — Audit & Compliance Context](#8-nxa_audit--audit--compliance-context)
9. [nxa_core — Core Platform Context](#9-nxa_core--core-platform-context)
10. [Cross-Context Integration Events](#10-cross-context-integration-events)
11. [Domain Services & Specifications](#11-domain-services--specifications)
12. [Invariants & Business Rules (Domain Level)](#12-invariants--business-rules-domain-level)
13. [Diagrams](#13-diagrams)
14. [Tables](#14-tables)

---

## 1. Bounded Context Map

### 1.1 Context Diagram (Strategic)

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    BOUNDED CONTEXT MAP                                              │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│   ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│   │   nxa_accounts  │    │   nxa_tenants   │    │   nxa_assets    │    │ nxa_lansweeper  │         │
│   │  (Identity)     │◀──▶│  (Tenancy)      │◀──▶│  (Asset Core)   │◀──▶│  (Integration)  │         │
│   └────────┬────────┘    └────────┬────────┘    └────────┬────────┘    └────────┬────────┘         │
│            │                      │                      │                      │                   │
│            │            ┌─────────┴─────────┐            │                      │                   │
│            │            │                   │            │                      │                   │
│            ▼            ▼                   ▼            ▼                      ▼                   │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────┐      │
│   │                              nxa_core (Core Platform)                                   │      │
│   │         API Gateway │ Feature Flags │ Audit Log │ Developer Portal │ Background Jobs    │      │
│   └─────────────────────────────────────────────────────────────────────────────────────────┘      │
│            ▲                      ▲                      ▲                      ▲                   │
│            │                      │                      │                      │                   │
│   ┌────────┴────────┐    ┌────────┴────────┐    ┌────────┴────────┐    ┌────────┴────────┐         │
│   │ nxa_billing     │    │ nxa_notifications│    │ nxa_audit       │    │  (External)     │         │
│   │ (Billing)       │    │ (Notifications)  │    │ (Compliance)    │    │  Lansweeper     │         │
│   └─────────────────┘    └─────────────────┘    └─────────────────┘    │  Cloud Providers│         │
│                                                                         │  ITSM/SIEM      │         │
│                                                                         └─────────────────┘         │
│                                                                                                     │
│   RELATIONSHIPS:                                                                                    │
│   ┌─────────────┬──────────────────────────────────────────────────────────────────────────────┐   │
│   │ Pattern     │ Contexts                                                                     │   │
│   ├─────────────┼──────────────────────────────────────────────────────────────────────────────┤   │
│   │ Partnership │ nxa_accounts ↔ nxa_tenants (mutual dependency for authz)                    │   │
│   │ Customer/   │ nxa_assets (C) → nxa_lansweeper (S) — assets consumes discovery             │   │
│   │ Supplier    │ nxa_billing (C) → nxa_assets (S) — billing consumes asset data              │   │
│   │ Customer/   │ nxa_audit (C) → all (S) — audit observes all domains                        │   │
│   │ Supplier    │ nxa_notifications (S) → all (C) — notifications publishes events            │   │
│   │ Conformist  │ nxa_lansweeper → nxa_assets (anti-corruption layer)                         │   │
│   └─────────────┴──────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Context Responsibilities

| Context | Package | Primary Responsibility | Key Aggregates | Upstream Dependencies |
|---------|---------|------------------------|----------------|----------------------|
| **nxa_accounts** | `backend.nexusasset.nxa_accounts` | Authentication, authorization, identity lifecycle | `User`, `Role`, `ApiKey`, `Session` | — |
| **nxa_tenants** | `backend.nexusasset.nxa_tenants` | Tenant lifecycle, provisioning, quotas, hierarchy | `Tenant`, `TenantSettings`, `TenantUser` | `nxa_accounts` |
| **nxa_assets** | `backend.nexusasset.nxa_assets` | Asset lifecycle, CMDB, relationships, maintenance | `Asset`, `CIRelationship`, `MaintenanceWindow` | `nxa_tenants`, `nxa_accounts` |
| **nxa_lansweeper** | `backend.nexusasset.nxa_lansweeper` | Lansweeper sync, reconciliation, field ownership | `LansweeperSite`, `SyncJob`, `ReconciliationRule` | `nxa_assets` (ACL) |
| **nxa_billing** | `backend.nexusasset.nxa_billing` | Subscriptions, invoicing, cost allocation, contracts | `Subscription`, `Invoice`, `AllocationRule`, `Contract` | `nxa_tenants`, `nxa_assets` |
| **nxa_notifications** | `backend.nexusasset.nxa_notifications` | Channel management, rule engine, delivery | `NotificationChannel`, `NotificationRule`, `Notification` | `nxa_tenants`, `nxa_accounts` |
| **nxa_audit** | `backend.nexusasset.nxa_audit` | Compliance frameworks, assessments, evidence, risk | `Framework`, `Control`, `Assessment`, `EvidencePackage`, `Risk` | All (observer) |
| **nxa_core** | `backend.nexusasset.nxa_core` | Platform services: gateway, flags, audit log, jobs | `FeatureFlag`, `AuditLogEntry`, `BackgroundJob` | All (infrastructure) |

---

## 2. nxa_accounts — Identity & Access Context

### 2.1 Aggregate: User (AGG-AC-001)

**Root Entity**: `User` (ENT-AC-001)

```python
# Domain Model (Pure Python - no Django/ORM)
@dataclass(frozen=True)
class UserId:
    value: UUID  # UUID v7

@dataclass(frozen=True)
class Email:
    value: str  # Validated email format

@dataclass(frozen=True)
class PasswordHash:
    value: str  # Argon2id hash

class User(AggregateRoot[UserId]):
    id: UserId
    email: Email
    password_hash: PasswordHash
    display_name: str
    timezone: str  # IANA tz
    locale: str    # BCP 47
    mfa_enabled: bool
    mfa_methods: List[MfaMethod]  # VO-AC-001
    status: UserStatus  # ACTIVE, SUSPENDED, PENDING_VERIFICATION, DELETED
    created_at: DateTime
    updated_at: DateTime
    deleted_at: Optional[DateTime]
    tenant_id: TenantId  # FK to nxa_tenants
    
    # Domain Methods (invariants enforced)
    def enable_mfa(self, method: MfaMethod) -> None:
        """Enforce: max 5 methods, at least one TOTP or WebAuthn"""
    
    def disable_mfa(self, method_id: MfaMethodId, actor: UserId) -> None:
        """Enforce: cannot disable last MFA method if required by tenant policy"""
    
    def suspend(self, reason: str, actor: UserId) -> None:
        """Enforce: only tenant admin or platform admin"""
    
    def add_api_key(self, key: ApiKey) -> None:
        """Enforce: max 50 keys per user"""
    
    def revoke_api_key(self, key_id: ApiKeyId) -> None:
        pass
```

**Entities within Aggregate**:
- `ApiKey` (ENT-AC-002) — scoped to user, hashed at rest
- `Session` (ENT-AC-003) — device, IP, user agent, expiry
- `MfaMethod` (ENT-AC-004) — TOTP, WebAuthn, backup codes

**Value Objects**:
- `UserId` (VO-AC-001) — UUID v7
- `Email` (VO-AC-002) — validated format
- `PasswordHash` (VO-AC-003) — Argon2id
- `MfaMethod` (VO-AC-004) — type, secret (encrypted), label, registered_at
- `ApiKeyId` (VO-AC-005) — UUID v7
- `ApiKeyHash` (VO-AC-006) — bcrypt
- `SessionId` (VO-AC-007) — UUID v7

### 2.2 Aggregate: Role (AGG-AC-002)

**Root Entity**: `Role` (ENT-AC-005)

```python
class Role(AggregateRoot[RoleId]):
    id: RoleId
    tenant_id: TenantId
    name: str  # Unique per tenant
    description: str
    permissions: Set[Permission]  # VO-AC-007
    is_system: bool  # Platform roles (immutable)
    is_custom: bool  # Tenant-created
    parent_role_id: Optional[RoleId]  # Inheritance
    created_at: DateTime
    updated_at: DateTime
    
    def add_permission(self, permission: Permission) -> None:
        """Enforce: cannot add platform-only permissions to custom roles"""
    
    def remove_permission(self, permission: Permission) -> None:
        pass
    
    def set_parent(self, parent: RoleId) -> None:
        """Enforce: no circular inheritance, depth ≤ 5"""
```

**Value Objects**:
- `RoleId` (VO-AC-008) — UUID v7
- `Permission` (VO-AC-009) — `resource:action:scope` (e.g., `assets:read:own`, `assets:write:*`)

### 2.3 Domain Events (EVT-AC-###)

| Event ID | Name | Payload | Trigger |
|----------|------|---------|---------|
| **EVT-AC-001** | `UserRegistered` | `user_id, email, tenant_id, method` | User creation |
| **EVT-AC-002** | `UserAuthenticated` | `user_id, tenant_id, method, mfa_used, ip` | Successful login |
| **EVT-AC-003** | `UserMfaEnabled` | `user_id, method_type, method_id` | MFA registration |
| **EVT-AC-004** | `UserMfaDisabled` | `user_id, method_type, method_id` | MFA removal |
| **EVT-AC-005** | `ApiKeyCreated` | `user_id, key_id, scopes, expires_at` | API key generation |
| **EVT-AC-006** | `ApiKeyRevoked` | `user_id, key_id, reason` | API key revocation |
| **EVT-AC-007** | `RoleAssigned` | `user_id, role_id, tenant_id, assigned_by` | RBAC assignment |
| **EVT-AC-008** | `RoleRevoked` | `user_id, role_id, tenant_id, revoked_by` | RBAC removal |
| **EVT-AC-009** | `SessionCreated` | `user_id, session_id, device_info, ip` | New session |
| **EVT-AC-010** | `SessionRevoked` | `user_id, session_id, reason` | Logout/revocation |
| **EVT-AC-011** | `ImpersonationStarted` | `actor_id, target_id, tenant_id, expires_at` | Support impersonation |
| **EVT-AC-012** | `ImpersonationEnded` | `actor_id, target_id, duration_seconds` | Impersonation end |

---

## 3. nxa_tenants — Tenant Management Context

### 3.1 Aggregate: Tenant (AGG-TN-001)

**Root Entity**: `Tenant` (ENT-TN-001)

```python
class Tenant(AggregateRoot[TenantId]):
    id: TenantId
    name: str
    slug: str  # Unique, DNS-safe, used for subdomain
    status: TenantStatus  # PROVISIONING, ACTIVE, SUSPENDED, OFFBOARDING, OFFBOARDED
    plan: PlanTier  # FREE, PRO, ENTERPRISE, CUSTOM
    billing_email: Email
    primary_domain: Optional[DomainName]  # Custom domain
    branding: BrandingSettings  # VO-TN-001
    settings: TenantSettings  # VO-TN-002
    quotas: QuotaLimits  # VO-TN-003
    region: Region  # VO-TN-004 (data residency)
    parent_tenant_id: Optional[TenantId]  # For hierarchy
    created_at: DateTime
    activated_at: Optional[DateTime]
    suspended_at: Optional[DateTime]
    offboarded_at: Optional[DateTime]
    legal_hold: bool
    
    def provision(self) -> None:
        """Enforce: creates schema, RLS policies, default roles, branding"""
    
    def suspend(self, reason: str, actor: UserId) -> None:
        """Enforce: only platform admin; revokes all sessions"""
    
    def reactivate(self, actor: UserId) -> None:
        """Enforce: only platform admin; grace period check"""
    
    def initiate_offboarding(self, actor: UserId) -> None:
        """Enforce: 30-day grace period unless legal_hold; schedules deletion"""
    
    def cancel_offboarding(self, actor: UserId) -> None:
        """Enforce: only during grace period"""
    
    def complete_offboarding(self) -> None:
        """Enforce: cryptographic erase, schema drop, certificate issuance"""
    
    def update_quotas(self, quotas: QuotaLimits, actor: UserId) -> None:
        """Enforce: platform admin only; validates against plan limits"""
    
    def set_custom_domain(self, domain: DomainName, actor: UserId) -> None:
        """Enforce: CNAME verification, TLS provisioning"""
```

**Value Objects**:
- `TenantId` (VO-TN-001) — UUID v7
- `BrandingSettings` (VO-TN-002) — logo_url, primary_color, favicon_url, custom_css
- `TenantSettings` (VO-TN-003) — custom_fields, workflows, retention_policies, integration_configs
- `QuotaLimits` (VO-TN-004) — max_assets, max_api_calls_per_min, max_storage_gb, max_concurrent_scans, max_seats
- `Region` (VO-TN-005) — code (eu-west-1, us-east-1), display_name, data_residency_required
- `DomainName` (VO-TN-006) — validated FQDN
- `PlanTier` (VO-TN-007) — enum with default quotas

### 3.2 Aggregate: TenantUser (AGG-TN-002)

**Root Entity**: `TenantUser` (ENT-TN-002) — Link between User and Tenant with tenant-specific roles

```python
class TenantUser(AggregateRoot[TenantUserId]):
    id: TenantUserId
    tenant_id: TenantId
    user_id: UserId
    roles: Set[RoleId]  # Tenant-scoped roles
    groups: Set[GroupId]
    status: TenantUserStatus  # ACTIVE, INVITED, SUSPENDED, REMOVED
    invited_at: Optional[DateTime]
    joined_at: Optional[DateTime]
    removed_at: Optional[DateTime]
    invited_by: Optional[UserId]
    
    def assign_role(self, role_id: RoleId, actor: UserId) -> None:
        """Enforce: role belongs to same tenant; no platform roles"""
    
    def remove_role(self, role_id: RoleId, actor: UserId) -> None:
        pass
```

### 3.3 Domain Events (EVT-TN-###)

| Event ID | Name | Payload | Trigger |
|----------|------|---------|---------|
| **EVT-TN-001** | `TenantProvisioned` | `tenant_id, name, slug, region, plan` | Provisioning complete |
| **EVT-TN-002** | `TenantActivated` | `tenant_id, activated_at` | First admin login |
| **EVT-TN-003** | `TenantSuspended` | `tenant_id, reason, suspended_by` | Suspension |
| **EVT-TN-004** | `TenantReactivated` | `tenant_id, reactivated_by` | Reactivation |
| **EVT-TN-005** | `TenantOffboardingInitiated` | `tenant_id, grace_period_end, initiated_by` | Offboarding start |
| **EVT-TN-006** | `TenantOffboardingCancelled` | `tenant_id, cancelled_by` | Grace period cancel |
| **EVT-TN-007** | `TenantOffboarded` | `tenant_id, certificate_id, offboarded_at` | Secure deletion complete |
| **EVT-TN-008** | `TenantQuotasUpdated` | `tenant_id, old_quotas, new_quotas, updated_by` | Quota change |
| **EVT-TN-009** | `CustomDomainConfigured` | `tenant_id, domain, verified_at` | Domain verified |
| **EVT-TN-010** | `TenantUserInvited` | `tenant_id, user_id, email, roles, invited_by` | Invitation sent |
| **EVT-TN-011** | `TenantUserJoined` | `tenant_id, user_id, joined_at` | Invitation accepted |
| **EVT-TN-012** | `TenantUserRemoved` | `tenant_id, user_id, removed_by` | User removal |

---

## 4. nxa_assets — Asset Core Context

### 4.1 Aggregate: Asset (AGG-AS-001)

**Root Entity**: `Asset` (ENT-AS-001) — The central aggregate for all asset types

```python
class Asset(AggregateRoot[AssetId]):
    id: AssetId
    tenant_id: TenantId
    
    # Identification
    asset_type: AssetType  # SERVER, LAPTOP, NETWORK_DEVICE, VM, CONTAINER, CLOUD_RESOURCE, SOFTWARE, PERIPHERAL, OTHER
    subtype: Optional[str]  # e.g., "EC2", "K8s_Pod", "Dell_R750"
    name: str
    serial_number: Optional[SerialNumber]  # VO-AS-001
    asset_tag: Optional[AssetTag]  # VO-AS-002 (barcode/QR)
    
    # Hardware Specs (for physical/virtual)
    manufacturer: Optional[str]
    model: Optional[str]
    cpu: Optional[CpuSpec]  # VO-AS-003
    memory: Optional[MemorySpec]  # VO-AS-004
    storage: List[StorageSpec]  # VO-AS-005
    network_interfaces: List[NetworkInterface]  # VO-AS-006
    
    # Cloud Identity (for cloud resources)
    cloud_provider: Optional[CloudProvider]  # AWS, AZURE, GCP, OTHER
    cloud_resource_id: Optional[str]  # ARN, Azure Resource ID, GCP selfLink
    cloud_region: Optional[str]
    cloud_account_id: Optional[str]
    cloud_tags: Dict[str, str]  # Provider tags
    
    # Software
    installed_software: List[SoftwareInstallation]  # ENT-AS-002 (nested entity)
    
    # Ownership & Location
    owner_id: Optional[UserId]  # Assigned user
    custodian_id: Optional[UserId]  # Responsible party
    location_id: Optional[LocationId]  # Physical/logical location
    rack_position: Optional[RackPosition]  # VO-AS-007
    geo_coordinates: Optional[GeoCoordinates]  # VO-AS-008
    
    # Lifecycle
    state: AssetState  # DISCOVERED, AUTHORIZED, ACTIVE, MAINTENANCE, RETIRED, DISPOSED
    state_changed_at: DateTime
    state_changed_by: UserId
    state_reason: Optional[str]
    
    # Financial
    purchase_date: Optional[Date]
    purchase_cost: Optional[Money]  # VO-AS-009
    depreciation_method: DepreciationMethod  # STRAIGHT_LINE, DECLINING_BALANCE
    useful_life_months: Optional[int]
    warranty_end: Optional[Date]
    support_contract_id: Optional[str]
    
    # Metadata
    tags: Set[Tag]  # VO-AS-010
    custom_fields: Dict[str, CustomFieldValue]  # VO-AS-011
    sources: Set[AssetSource]  # VO-AS-012 (LANSWEEPER, CLOUD_API, MANUAL, AGENT)
    confidence_score: float  # 0.0-1.0 (reconciliation confidence)
    last_seen_at: Optional[DateTime]  # Last discovery timestamp
    created_at: DateTime
    updated_at: DateTime
    created_by: UserId
    
    # Domain Methods (invariants)
    def authorize(self, actor: UserId, reason: str) -> None:
        """Transition: DISCOVERED → AUTHORIZED. Enforce: policy check, quota"""
    
    def activate(self, actor: UserId, owner_id: UserId, location_id: LocationId) -> None:
        """Transition: AUTHORIZED → ACTIVE. Enforce: assignment required"""
    
    def start_maintenance(self, actor: UserId, window: MaintenanceWindow) -> None:
        """Transition: ACTIVE → MAINTENANCE. Enforce: SLA tracking starts"""
    
    def complete_maintenance(self, actor: UserId, outcome: MaintenanceOutcome) -> None:
        """Transition: MAINTENANCE → ACTIVE. Enforce: outcome recorded"""
    
    def retire(self, actor: UserId, reason: str, sanitization_cert: SanitizationCertificate) -> None:
        """Transition: ACTIVE/MAINTENANCE → RETIRED. Enforce: NIST 800-88 cert"""
    
    def dispose(self, actor: UserId, disposal_cert: DisposalCertificate) -> None:
        """Transition: RETIRED → DISPOSED. Enforce: financial write-off"""
    
    def restore(self, actor: UserId, reason: str) -> None:
        """Transition: RETIRED → ACTIVE. Enforce: re-certification"""
    
    def update_hardware_specs(self, specs: HardwareSpecs, source: AssetSource, actor: UserId) -> None:
        """Enforce: Field Ownership Matrix - Lansweeper wins for HW"""
    
    def update_cloud_identity(self, identity: CloudIdentity, source: AssetSource, actor: UserId) -> None:
        """Enforce: Field Ownership Matrix - Cloud API wins for cloud tags"""
    
    def update_ownership(self, owner_id: UserId, custodian_id: UserId, actor: UserId) -> None:
        """Enforce: Field Ownership Matrix - Manual wins for owner/location"""
    
    def add_relationship(self, target_id: AssetId, rel_type: CiRelationshipType, actor: UserId) -> None:
        """Enforce: no circular depends_on; cardinality limits"""
    
    def remove_relationship(self, target_id: AssetId, rel_type: CiRelationshipType, actor: UserId) -> None:
        pass
```

**Nested Entities** (within Asset aggregate):
- `SoftwareInstallation` (ENT-AS-002) — name, version, publisher, install_date, license_key, license_type

**Value Objects**:
- `AssetId` (VO-AS-001) — UUID v7
- `SerialNumber` (VO-AS-002) — alphanumeric, max 100 chars
- `AssetTag` (VO-AS-003) — barcode/QR format, unique per tenant
- `CpuSpec` (VO-AS-004) — model, cores, threads, frequency_ghz, architecture
- `MemorySpec` (VO-AS-005) — total_gb, type (DDR4/DDR5), speed_mhz
- `StorageSpec` (VO-AS-006) — device, capacity_gb, type (SSD/HDD/NVMe), interface
- `NetworkInterface` (VO-AS-007) — name, mac_address, ip_addresses, speed_gbps, vlan
- `RackPosition` (VO-AS-008) — datacenter, room, rack, u_position, u_height
- `GeoCoordinates` (VO-AS-009) — latitude, longitude, accuracy_meters
- `Money` (VO-AS-010) — amount (Decimal), currency (ISO 4217)
- `Tag` (VO-AS-011) — key:value, tenant-scoped
- `CustomFieldValue` (VO-AS-012) — typed value per custom field definition
- `AssetSource` (VO-AS-013) — enum: LANSWEEPER, CLOUD_API, MANUAL, AGENT, IMPORT
- `HardwareSpecs` (VO-AS-014) — cpu, memory, storage, network_interfaces
- `CloudIdentity` (VO-AS-015) — provider, resource_id, region, account_id, tags
- `SanitizationCertificate` (VO-AS-016) — method (NIST_800_88_PURGE/CLEAR/DESTROY), verified_by, verified_at, certificate_hash
- `DisposalCertificate` (VO-AS-017) — vendor, certificate_number, disposed_at, environmental_compliance
- `MaintenanceWindow` (VO-AS-018) — start, end, technician_id, type (PREVENTIVE/CORRECTIVE)
- `MaintenanceOutcome` (VO-AS-019) — status, actual_start, actual_end, parts_used, labor_hours, notes

### 4.2 Aggregate: CIRelationship (AGG-AS-002)

**Root Entity**: `CIRelationship` (ENT-AS-003) — Explicit relationship entity for graph operations

```python
class CIRelationship(AggregateRoot[CiRelationshipId]):
    id: CiRelationshipId
    tenant_id: TenantId
    source_id: AssetId
    target_id: AssetId
    relationship_type: CiRelationshipType  # DEPENDS_ON, HOSTS, CONNECTS_TO, CONTAINS, RUNS_ON, MANAGES
    metadata: Dict[str, str]  # e.g., port, protocol, dependency_strength
    created_at: DateTime
    created_by: UserId
    
    def validate_acyclic(self, repository: CiRelationshipRepository) -> None:
        """Enforce: DEPENDS_ON graph must be acyclic"""
```

**Value Objects**:
- `CiRelationshipId` (VO-AS-020) — UUID v7
- `CiRelationshipType` (VO-AS-021) — enum with cardinality constraints

### 4.3 Aggregate: MaintenanceWindow (AGG-AS-003)

**Root Entity**: `MaintenanceWindow` (ENT-AS-004)

```python
class MaintenanceWindow(AggregateRoot[MaintenanceWindowId]):
    id: MaintenanceWindowId
    tenant_id: TenantId
    asset_id: AssetId
    title: str
    description: str
    window: MaintenanceWindow  # VO-AS-018 (reused)
    recurrence: Optional[RecurrenceRule]  # VO-AS-022 (RRULE)
    sla_target: Duration  # e.g., 4 hours
    status: MaintenanceStatus  # SCHEDULED, IN_PROGRESS, COMPLETED, FAILED, CANCELLED
    outcome: Optional[MaintenanceOutcome]  # VO-AS-019
    assigned_technician_id: Optional[UserId]
    assigned_team_id: Optional[TeamId]
    created_at: DateTime
    updated_at: DateTime
    created_by: UserId
```

### 4.4 Domain Events (EVT-AS-###)

| Event ID | Name | Payload | Trigger |
|----------|------|---------|---------|
| **EVT-AS-001** | `AssetDiscovered` | `asset_id, tenant_id, source, raw_data_hash` | New asset from discovery |
| **EVT-AS-002** | `AssetReconciled` | `asset_id, tenant_id, merge_decisions, confidence` | Reconciliation complete |
| **EVT-AS-003** | `AssetAuthorized` | `asset_id, tenant_id, authorized_by` | State: DISCOVERED→AUTHORIZED |
| **EVT-AS-004** | `AssetActivated` | `asset_id, tenant_id, owner_id, location_id, activated_by` | State: AUTHORIZED→ACTIVE |
| **EVT-AS-005** | `AssetStateChanged` | `asset_id, tenant_id, from_state, to_state, reason, changed_by` | Any state transition |
| **EVT-AS-006** | `AssetAssigned` | `asset_id, tenant_id, owner_id, custodian_id, assigned_by` | Ownership change |
| **EVT-AS-007** | `AssetRelocated` | `asset_id, tenant_id, from_location, to_location, relocated_by` | Location change |
| **EVT-AS-008** | `MaintenanceStarted` | `window_id, asset_id, tenant_id, technician_id` | Maintenance begin |
| **EVT-AS-009** | `MaintenanceCompleted` | `window_id, asset_id, tenant_id, outcome` | Maintenance end |
| **EVT-AS-010** | `AssetRetired` | `asset_id, tenant_id, reason, sanitization_cert_id, retired_by` | State: →RETIRED |
| **EVT-AS-011** | `AssetDisposed` | `asset_id, tenant_id, disposal_cert_id, disposed_by` | State: →DISPOSED |
| **EVT-AS-012** | `CiRelationshipCreated` | `relationship_id, source_id, target_id, type, created_by` | Relationship added |
| **EVT-AS-013** | `CiRelationshipRemoved` | `relationship_id, removed_by` | Relationship removed |
| **EVT-AS-014** | `SoftwareInstalled` | `asset_id, software_id, name, version, license_type` | Software discovered |
| **EVT-AS-015** | `AssetComparisonGenerated` | `asset_id, version_a, version_b, diff_summary, generated_by` | Diff requested |

---

## 5. nxa_lansweeper — Lansweeper Integration Context

### 5.1 Aggregate: LansweeperSite (AGG-LS-001)

**Root Entity**: `LansweeperSite` (ENT-LS-001)

```python
class LansweeperSite(AggregateRoot[LansweeperSiteId]):
    id: LansweeperSiteId
    tenant_id: TenantId
    name: str
    graphql_endpoint: Url  # VO-LS-001
    auth_config: LansweeperAuthConfig  # VO-LS-002 (PAT or OAuth2)
    sync_schedule: CronExpression  # VO-LS-003
    asset_type_filters: List[AssetType]  # Which types to sync
    field_ownership_rules: List[FieldOwnershipRule]  # VO-LS-004
    webhook_secret: Optional[Secret]  # VO-LS-005 (for signature verification)
    status: SiteStatus  # ACTIVE, PAUSED, ERROR, CONFIGURING
    last_sync_at: Optional[DateTime]
    last_sync_status: Optional[SyncStatus]  # SUCCESS, PARTIAL, FAILED
    last_sync_stats: Optional[SyncStatistics]  # VO-LS-006
    created_at: DateTime
    updated_at: DateTime
    created_by: UserId
    
    def update_schedule(self, schedule: CronExpression, actor: UserId) -> None:
        pass
    
    def update_field_ownership(self, rules: List[FieldOwnershipRule], actor: UserId) -> None:
        """Enforce: rules cover all syncable fields; no conflicts"""
    
    def record_sync_result(self, stats: SyncStatistics, status: SyncStatus) -> None:
        pass
```

### 5.2 Aggregate: SyncJob (AGG-LS-002)

**Root Entity**: `SyncJob` (ENT-LS-002)

```python
class SyncJob(AggregateRoot[SyncJobId]):
    id: SyncJobId
    tenant_id: TenantId
    site_id: LansweeperSiteId
    trigger: SyncTrigger  # SCHEDULED, MANUAL, WEBHOOK, RETRY
    sync_type: SyncType  # FULL, INCREMENTAL
    status: SyncJobStatus  # PENDING, RUNNING, COMPLETED, FAILED, CANCELLED
    started_at: Optional[DateTime]
    completed_at: Optional[DateTime]
    assets_processed: int
    assets_created: int
    assets_updated: int
    assets_deleted: int
    assets_failed: int
    errors: List[SyncError]  # VO-LS-007
    cursor: Optional[str]  # For pagination/resume
    idempotency_key: IdempotencyKey  # VO-LS-008
    created_at: DateTime
    created_by: Optional[UserId]  # None if scheduled
```

### 5.3 Aggregate: ReconciliationRule (AGG-LS-003)

**Root Entity**: `ReconciliationRule` (ENT-LS-003) — Field Ownership Matrix entry

```python
class ReconciliationRule(AggregateRoot[ReconciliationRuleId]):
    id: ReconciliationRuleId
    tenant_id: TenantId
    field_path: str  # Dot notation: "hardware.cpu.model", "cloud.tags.Environment"
    source_priority: List[AssetSource]  # Ordered: [LANSWEEPER, CLOUD_API, MANUAL]
    conflict_resolution: ConflictResolution  # SOURCE_WINS, LAST_WRITE_WINS, MERGE, MANUAL_REVIEW
    lock_field: bool  # Prevent overwrite once set manually
    merge_strategy: Optional[MergeStrategy]  # For MERGE: CONCATENATE, UNION, CUSTOM
    applies_to_types: List[AssetType]  # Empty = all types
    created_at: DateTime
    updated_at: DateTime
    created_by: UserId
```

### 5.4 Value Objects

| VO ID | Name | Description |
|-------|------|-------------|
| **VO-LS-001** | `Url` | Validated HTTP/HTTPS URL |
| **VO-LS-002** | `LansweeperAuthConfig` | type (PAT/OAUTH2), credentials (encrypted), token_url, client_id |
| **VO-LS-003** | `CronExpression` | Validated cron with timezone |
| **VO-LS-004** | `FieldOwnershipRule` | field_path, source_priority, conflict_resolution, lock_field, merge_strategy, applies_to_types |
| **VO-LS-005** | `Secret` | Encrypted secret value (Vault reference) |
| **VO-LS-006** | `SyncStatistics` | processed, created, updated, deleted, failed, duration_ms, throughput_per_sec |
| **VO-LS-007** | `SyncError` | asset_external_id, field_path, error_code, message, severity (WARNING/ERROR) |
| **VO-LS-008** | `IdempotencyKey` | UUID v7 for exactly-once processing |

### 5.5 Domain Events (EVT-LS-###)

| Event ID | Name | Payload | Trigger |
|----------|------|---------|---------|
| **EVT-LS-001** | `LansweeperSiteConfigured` | `site_id, tenant_id, endpoint, schedule` | Site creation |
| **EVT-LS-002** | `SyncJobStarted` | `job_id, site_id, tenant_id, trigger, sync_type` | Sync begins |
| **EVT-LS-003** | `SyncJobCompleted` | `job_id, site_id, tenant_id, stats, status` | Sync ends |
| **EVT-LS-004** | `SyncJobFailed` | `job_id, site_id, tenant_id, errors, retryable` | Sync failure |
| **EVT-LS-005** | `FieldOwnershipUpdated` | `site_id, tenant_id, rules_count, updated_by` | Matrix change |
| **EVT-LS-006** | `ReconciliationConflictDetected` | `conflict_id, asset_id, field_path, source_values, confidence` | Conflict found |
| **EVT-LS-007** | `ReconciliationConflictResolved` | `conflict_id, resolution, resolved_by, chosen_value` | Conflict resolved |
| **EVT-LS-008** | `WebhookReceived` | `site_id, tenant_id, event_type, asset_external_id, payload_hash` | Lansweeper webhook |
| **EVT-LS-009** | `DeltaSyncProcessed` | `site_id, tenant_id, assets_changed, duration_ms` | Incremental sync |

---

## 6. nxa_billing — Billing & Contracts Context

### 6.1 Aggregate: Subscription (AGG-BL-001)

**Root Entity**: `Subscription` (ENT-BL-001)

```python
class Subscription(AggregateRoot[SubscriptionId]):
    id: SubscriptionId
    tenant_id: TenantId
    plan_id: PlanId
    status: SubscriptionStatus  # TRIAL, ACTIVE, PAST_DUE, CANCELLED, EXPIRED, PAUSED
    billing_cycle: BillingCycle  # MONTHLY, ANNUAL
    current_period_start: DateTime
    current_period_end: DateTime
    trial_end: Optional[DateTime]
    quantity: int  # Seats, assets, etc. per plan
    unit_price: Money  # VO-AS-009 (reused)
    currency: Currency  # ISO 4217
    payment_method_id: Optional[PaymentMethodId]
    stripe_subscription_id: Optional[str]  # External reference
    metadata: Dict[str, str]
    created_at: DateTime
    updated_at: DateTime
    cancelled_at: Optional[DateTime]
    cancelled_by: Optional[UserId]
    
    def change_plan(self, new_plan_id: PlanId, proration: ProrationPolicy, actor: UserId) -> None:
        """Enforce: immediate for upgrade, period-end for downgrade"""
    
    def cancel(self, actor: UserId, at_period_end: bool = True) -> None:
        pass
    
    def pause(self, actor: UserId) -> None:
        """Enforce: max 3 months, enterprise only"""
    
    def resume(self, actor: UserId) -> None:
        pass
    
    def update_quantity(self, quantity: int, actor: UserId) -> None:
        """Enforce: within plan limits"""
```

### 6.2 Aggregate: Invoice (AGG-BL-002)

**Root Entity**: `Invoice` (ENT-BL-002)

```python
class Invoice(AggregateRoot[InvoiceId]):
    id: InvoiceId
    tenant_id: TenantId
    subscription_id: Optional[SubscriptionId]
    invoice_number: str  # Sequential per tenant
    status: InvoiceStatus  # DRAFT, OPEN, PAID, VOIDED, UNCOLLECTIBLE
    billing_period_start: DateTime
    billing_period_end: DateTime
    line_items: List[InvoiceLineItem]  # ENT-BL-003 (nested)
    subtotal: Money
    tax: Money
    total: Money
    currency: Currency
    due_date: DateTime
    paid_at: Optional[DateTime]
    payment_intent_id: Optional[str]  # Stripe reference
    pdf_url: Optional[str]  # Signed S3 URL
    idempotency_key: IdempotencyKey  # VO-LS-008 (reused)
    created_at: DateTime
    updated_at: DateTime
    
    def mark_paid(self, payment_intent_id: str, paid_at: DateTime) -> None:
        """Enforce: idempotent; only from OPEN"""
    
    def void(self, reason: str, actor: UserId) -> None:
        """Enforce: only DRAFT or OPEN; creates credit note"""
```

**Nested Entity**: `InvoiceLineItem` (ENT-BL-003)

```python
class InvoiceLineItem(Entity[InvoiceLineItemId]):
    id: InvoiceLineItemId
    description: str
    quantity: Decimal
    unit_price: Money
    tax_rate: Decimal  # e.g., 0.20 for 20% VAT
    tax_amount: Money
    total: Money
    metadata: Dict[str, str]  # asset_id, plan_id, usage_metric
```

### 6.3 Aggregate: AllocationRule (AGG-BL-003)

**Root Entity**: `AllocationRule` (ENT-BL-004)

```python
class AllocationRule(AggregateRoot[AllocationRuleId]):
    id: AllocationRuleId
    tenant_id: TenantId
    name: str
    dimension: AllocationDimension  # TENANT, DEPARTMENT, COST_CENTER, PROJECT, CUSTOM
    model: AllocationModel  # PROPORTIONAL_ASSETS, PROPORTIONAL_USERS, FIXED, TIERED, CUSTOM_FORMULA
    formula: Optional[str]  # JMESPath/CEL expression for CUSTOM_FORMULA
    allocations: List[AllocationTarget]  # VO-BL-001
    effective_from: Date
    effective_to: Optional[Date]
    is_active: bool
    created_at: DateTime
    updated_at: DateTime
    created_by: UserId
    
    def validate(self) -> None:
        """Enforce: allocations sum to 100% per period"""
```

### 6.4 Aggregate: Contract (AGG-BL-004)

**Root Entity**: `Contract` (ENT-BL-005)

```python
class Contract(AggregateRoot[ContractId]):
    id: ContractId
    tenant_id: TenantId
    vendor_name: str
    contract_type: ContractType  # SAAS, HARDWARE, SUPPORT, LICENSE, PROFESSIONAL_SERVICES
    title: str
    description: str
    start_date: Date
    end_date: Date
    value: Money
    currency: Currency
    renewal_type: RenewalType  # AUTO_RENEW, MANUAL_RENEW, NON_RENEWING
    notice_period_days: int
    obligations: List[ContractObligation]  # VO-BL-002
    documents: List[ContractDocument]  # VO-BL-003 (S3 refs)
    status: ContractStatus  # DRAFT, ACTIVE, EXPIRING, RENEWED, TERMINATED
    owner_id: UserId
    approvers: List[UserId]
    created_at: DateTime
    updated_at: DateTime
    
    def check_renewal(self) -> RenewalStatus:
        """Returns: UPCOMING, DUE, OVERDUE, COMPLETED"""
```

### 6.5 Value Objects

| VO ID | Name | Description |
|-------|------|-------------|
| **VO-BL-001** | `AllocationTarget` | dimension_value (e.g., dept_id), percentage (Decimal), cost_center_id |
| **VO-BL-002** | `ContractObligation` | description, frequency, due_date, responsible_party, status |
| **VO-BL-003** | `ContractDocument` | name, s3_key, content_type, size_bytes, uploaded_at, uploaded_by |
| **VO-BL-004** | `PlanId` | UUID v7 |
| **VO-BL-005** | `SubscriptionId` | UUID v7 |
| **VO-BL-006** | `InvoiceId` | UUID v7 |
| **VO-BL-007** | `AllocationRuleId` | UUID v7 |
| **VO-BL-008** | `ContractId` | UUID v7 |
| **VO-BL-009** | `PaymentMethodId` | UUID v7 |

### 6.6 Domain Events (EVT-BL-###)

| Event ID | Name | Payload | Trigger |
|----------|------|---------|---------|
| **EVT-BL-001** | `SubscriptionCreated` | `subscription_id, tenant_id, plan_id, billing_cycle, trial_end` | Purchase |
| **EVT-BL-002** | `SubscriptionChanged` | `subscription_id, tenant_id, old_plan, new_plan, proration` | Plan change |
| **EVT-BL-003** | `SubscriptionCancelled` | `subscription_id, tenant_id, at_period_end, cancelled_by` | Cancellation |
| **EVT-BL-004** | `InvoiceGenerated` | `invoice_id, tenant_id, subscription_id, total, due_date, idempotency_key` | Billing cycle |
| **EVT-BL-005** | `InvoicePaid` | `invoice_id, tenant_id, amount, payment_method, paid_at` | Payment success |
| **EVT-BL-006** | `InvoiceVoided` | `invoice_id, tenant_id, reason, voided_by` | Void |
| **EVT-BL-007** | `AllocationRuleCreated` | `rule_id, tenant_id, dimension, model` | Rule creation |
| **EVT-BL-008** | `AllocationExecuted` | `rule_id, tenant_id, period, allocations, total_allocated` | Monthly run |
| **EVT-BL-009** | `ContractCreated` | `contract_id, tenant_id, vendor, type, value, end_date` | Contract added |
| **EVT-BL-010** | `ContractRenewalDue` | `contract_id, tenant_id, days_until_renewal, owner_id` | 90/60/30 day alert |
| **EVT-BL-011** | `LicenseTrueUpGenerated` | `tenant_id, period, discrepancies, estimated_cost` | True-up run |

---

## 7. nxa_notifications — Notifications Context

### 7.1 Aggregate: NotificationChannel (AGG-NT-001)

**Root Entity**: `NotificationChannel` (ENT-NT-001)

```python
class NotificationChannel(AggregateRoot[NotificationChannelId]):
    id: NotificationChannelId
    tenant_id: TenantId
    name: str
    channel_type: ChannelType  # EMAIL, SMS, SLACK, TEAMS, WEBHOOK, PUSH, IN_APP
    config: ChannelConfig  # VO-NT-001 (type-specific config)
    templates: Dict[str, NotificationTemplate]  # VO-NT-002 (event_type -> template)
    is_active: bool
    is_default: bool
    rate_limit: Optional[RateLimit]  # VO-NT-003
    created_at: DateTime
    updated_at: DateTime
    created_by: UserId
    
    def test_delivery(self, test_payload: Dict) -> DeliveryResult:
        """Send test notification"""
```

### 7.2 Aggregate: NotificationRule (AGG-NT-002)

**Root Entity**: `NotificationRule` (ENT-NT-002)

```python
class NotificationRule(AggregateRoot[NotificationRuleId]):
    id: NotificationRuleId
    tenant_id: TenantId
    name: str
    description: str
    trigger_event: str  # Domain event type (e.g., "AssetStateChanged")
    conditions: ConditionExpression  # VO-NT-004 (JMESPath/CEL)
    channel_ids: List[NotificationChannelId]
    template_overrides: Dict[str, NotificationTemplate]  # Per-channel override
    frequency: NotificationFrequency  # IMMEDIATE, DIGEST_HOURLY, DIGEST_DAILY
    quiet_hours: Optional[QuietHours]  # VO-NT-005
    severity_filter: List[Severity]  # INFO, WARNING, ERROR, CRITICAL
    is_active: bool
    created_at: DateTime
    updated_at: DateTime
    created_by: UserId
    
    def evaluate(self, event: DomainEvent) -> bool:
        """Returns true if rule matches event"""
```

### 7.3 Aggregate: Notification (AGG-NT-003)

**Root Entity**: `Notification` (ENT-NT-003) — Individual notification instance

```python
class Notification(AggregateRoot[NotificationId]):
    id: NotificationId
    tenant_id: TenantId
    rule_id: Optional[NotificationRuleId]
    recipient_user_ids: List[UserId]  # Empty = broadcast to matching users
    channel_type: ChannelType
    subject: str
    body: str  # Rendered template
    priority: NotificationPriority  # LOW, NORMAL, HIGH, CRITICAL
    status: NotificationStatus  # PENDING, QUEUED, SENT, DELIVERED, FAILED, READ
    delivery_attempts: int
    last_attempt_at: Optional[DateTime]
    next_retry_at: Optional[DateTime]
    delivered_at: Optional[DateTime]
    read_at: Optional[DateTime]
    metadata: Dict[str, str]  # event_id, asset_id, correlation_id
    idempotency_key: IdempotencyKey
    created_at: DateTime
    
    def mark_sent(self, provider_message_id: str) -> None:
        pass
    
    def mark_delivered(self) -> None:
        pass
    
    def mark_failed(self, error: str, retryable: bool) -> None:
        """Enforce: max 5 retries with exponential backoff"""
    
    def mark_read(self, user_id: UserId) -> None:
        pass
```

### 7.4 Value Objects

| VO ID | Name | Description |
|-------|------|-------------|
| **VO-NT-001** | `ChannelConfig` | Email: smtp_host, from_address, credentials_ref; Slack: webhook_url, bot_token; Webhook: url, headers, secret |
| **VO-NT-002** | `NotificationTemplate` | subject_template (Handlebars), body_template (Handlebars), format (HTML/TEXT/MARKDOWN) |
| **VO-NT-003** | `RateLimit` | max_per_minute, max_per_hour, max_per_day |
| **VO-NT-004** | `ConditionExpression` | expression (JMESPath/CEL), language (JMESPath/CEL) |
| **VO-NT-005** | `QuietHours` | start_time (HH:MM), end_time (HH:MM), timezone (IANA) |

### 7.5 Domain Events (EVT-NT-###)

| Event ID | Name | Payload | Trigger |
|----------|------|---------|---------|
| **EVT-NT-001** | `NotificationChannelCreated` | `channel_id, tenant_id, type, is_default` | Channel setup |
| **EVT-NT-002** | `NotificationRuleCreated` | `rule_id, tenant_id, trigger_event, channel_count` | Rule creation |
| **EVT-NT-003** | `NotificationQueued` | `notification_id, tenant_id, channel_type, recipient_count` | Rule matched |
| **EVT-NT-004** | `NotificationSent` | `notification_id, tenant_id, provider_message_id, attempt` | Provider accepted |
| **EVT-NT-005** | `NotificationDelivered` | `notification_id, tenant_id, delivered_at` | Delivery confirmed |
| **EVT-NT-006** | `NotificationFailed` | `notification_id, tenant_id, error, retryable, attempt` | Delivery failed |
| **EVT-NT-007** | `NotificationRead` | `notification_id, tenant_id, user_id, read_at` | User read |

---

## 8. nxa_audit — Audit & Compliance Context

### 8.1 Aggregate: Framework (AGG-AU-001)

**Root Entity**: `Framework` (ENT-AU-001)

```python
class Framework(AggregateRoot[FrameworkId]):
    id: FrameworkId
    code: str  # ISO_27001_2022, SOC2_2017, GDPR_2016, HIPAA_1996, PCI_DSS_4_0, NIST_CSF_2_0, CIS_CONTROLS_V8
    name: str
    version: str
    description: str
    controls: List[Control]  # ENT-AU-002 (nested)
    is_active: bool
    created_at: DateTime
    updated_at: DateTime
```

### 8.2 Aggregate: Control (AGG-AU-002)

**Root Entity**: `Control` (ENT-AU-002) — Nested in Framework

```python
class Control(Entity[ControlId]):
    id: ControlId
    framework_id: FrameworkId
    control_id: str  # e.g., "A.8.1.1", "CC6.1"
    title: str
    description: str
    evidence_requirements: List[EvidenceRequirement]  # VO-AU-001
    testing_procedure: str
    frequency: AssessmentFrequency  # CONTINUOUS, DAILY, WEEKLY, MONTHLY, QUARTERLY, ANNUAL
    policy_refs: List[PolicyReference]  # VO-AU-002 (OPA policy IDs)
    remediation_guidance: str
    severity: ControlSeverity  # CRITICAL, HIGH, MEDIUM, LOW
```

### 8.3 Aggregate: Assessment (AGG-AU-003)

**Root Entity**: `Assessment` (ENT-AU-003)

```python
class Assessment(AggregateRoot[AssessmentId]):
    id: AssessmentId
    tenant_id: TenantId
    framework_id: FrameworkId
    name: str
    status: AssessmentStatus  # PLANNED, IN_PROGRESS, COMPLETED, FAILED, CANCELLED
    scope: AssessmentScope  # VO-AU-003 (filters: asset types, tags, locations)
    started_at: Optional[DateTime]
    completed_at: Optional[DateTime]
    findings: List[Finding]  # ENT-AU-004 (nested)
    overall_score: Optional[float]  # 0.0-100.0
    created_at: DateTime
    updated_at: DateTime
    created_by: UserId
    
    def execute(self, policy_engine: PolicyEngine) -> None:
        """Run all mapped policies, collect evidence, generate findings"""
    
    def generate_evidence_package(self) -> EvidencePackage:
        """Create immutable, signed package"""
```

### 8.4 Nested Entities

**Finding** (ENT-AU-004) — Within Assessment

```python
class Finding(Entity[FindingId]):
    id: FindingId
    assessment_id: AssessmentId
    control_id: ControlId
    resource_type: str  # Asset, User, Tenant, etc.
    resource_id: str
    status: FindingStatus  # PASS, FAIL, WARN, NOT_APPLICABLE, NOT_EVALUATED
    evidence_refs: List[EvidenceRef]  # VO-AU-004
    remediation: Optional[RemediationPlan]  # VO-AU-005
    risk_score: float  # 0.0-10.0 (CVSS-like)
    detected_at: DateTime
    resolved_at: Optional[DateTime]
```

**EvidencePackage** (ENT-AU-005) — Standalone aggregate

```python
class EvidencePackage(AggregateRoot[EvidencePackageId]):
    id: EvidencePackageId
    tenant_id: TenantId
    framework_id: FrameworkId
    control_id: str
    period_start: DateTime
    period_end: DateTime
    evidence_items: List[EvidenceItem]  # VO-AU-006
    merkle_root: str  # SHA-256
    signature: JwsSignature  # VO-AU-007 (RS256)
    chain_of_custody: List[CustodyEntry]  # VO-AU-008
    generated_at: DateTime
    generated_by: str  # System identifier
```

### 8.5 Aggregate: Risk (AGG-AU-004)

**Root Entity**: `Risk` (ENT-AU-006)

```python
class Risk(AggregateRoot[RiskId]):
    id: RiskId
    tenant_id: TenantId
    title: str
    description: str
    category: RiskCategory  # SECURITY, COMPLIANCE, OPERATIONAL, FINANCIAL, REPUTATIONAL, STRATEGIC
    likelihood: Likelihood  # RARE, UNLIKELY, POSSIBLE, LIKELY, ALMOST_CERTAIN (1-5)
    impact: Impact  # NEGLIGIBLE, MINOR, MODERATE, MAJOR, CATASTROPHIC (1-5)
    inherent_score: int  # likelihood × impact (1-25)
    treatment: RiskTreatment  # ACCEPT, MITIGATE, TRANSFER, AVOID
    residual_likelihood: Likelihood
    residual_impact: Impact
    residual_score: int
    owner_id: UserId
    mitigation_actions: List[MitigationAction]  # VO-AU-009
    review_date: Date
    status: RiskStatus  # IDENTIFIED, ASSESSED, TREATING, MONITORING, CLOSED
    created_at: DateTime
    updated_at: DateTime
```

### 8.6 Value Objects

| VO ID | Name | Description |
|-------|------|-------------|
| **VO-AU-001** | `EvidenceRequirement` | type (CONFIG_SNAPSHOT, LOG_EXPORT, SCAN_REPORT, POLICY_DOC), description, format |
| **VO-AU-002** | `PolicyReference` | policy_id (OPA bundle), version, parameters |
| **VO-AU-003** | `AssessmentScope` | asset_types, tags, locations, exclude_tags, custom_filter (JMESPath) |
| **VO-AU-004** | `EvidenceRef` | type, s3_key, hash, collected_at, collector |
| **VO-AU-005** | `RemediationPlan` | description, owner_id, due_date, status, verification_method |
| **VO-AU-006** | `EvidenceItem` | type, ref (s3_key), hash, description, collected_at |
| **VO-AU-007** | `JwsSignature` | algorithm (RS256), signature (base64), signing_key_id |
| **VO-AU-008** | `CustodyEntry` | handler, action, timestamp, signature |
| **VO-AU-009** | `MitigationAction` | description, owner_id, due_date, status, evidence_ref |

### 8.7 Domain Events (EVT-AU-###)

| Event ID | Name | Payload | Trigger |
|----------|------|---------|---------|
| **EVT-AU-001** | `AssessmentStarted` | `assessment_id, tenant_id, framework_id, scope` | Assessment run |
| **EVT-AU-002** | `AssessmentCompleted` | `assessment_id, tenant_id, overall_score, finding_counts` | Assessment done |
| **EVT-AU-003** | `FindingCreated` | `finding_id, assessment_id, control_id, resource, status, risk_score` | Policy eval fail |
| **EVT-AU-004** | `FindingResolved` | `finding_id, resolved_by, resolution_evidence` | Remediation done |
| **EVT-AU-005** | `EvidencePackageGenerated` | `package_id, tenant_id, framework_id, control_id, merkle_root` | Package created |
| **EVT-AU-006** | `RiskIdentified` | `risk_id, tenant_id, title, category, inherent_score, owner_id` | Risk registered |
| **EVT-AU-007** | `RiskTreatmentUpdated` | `risk_id, treatment, residual_score, updated_by` | Treatment change |
| **EVT-AU-008** | `IncidentCreated` | `incident_id, tenant_id, severity, title, detected_at` | Security incident |
| **EVT-AU-009** | `IncidentResolved` | `incident_id, resolved_at, root_cause, post_mortem_ref` | Incident closed |

---

## 9. nxa_core — Core Platform Context

### 9.1 Aggregate: FeatureFlag (AGG-CR-001)

**Root Entity**: `FeatureFlag` (ENT-CR-001)

```python
class FeatureFlag(AggregateRoot[FeatureFlagId]):
    id: FeatureFlagId
    key: str  # Unique, e.g., "new-asset-wizard"
    name: str
    description: str
    flag_type: FlagType  # BOOLEAN, STRING, NUMBER, JSON
    default_value: Any
    targeting_rules: List[TargetingRule]  # VO-CR-001
    variants: List[FlagVariant]  # VO-CR-002 (for multivariate)
    is_active: bool
    created_at: DateTime
    updated_at: DateTime
    created_by: UserId
    
    def evaluate(self, context: EvaluationContext) -> FlagValue:
        """Returns variant for given context (tenant, user, attributes)"""
```

### 9.2 Aggregate: AuditLogEntry (AGG-CR-002)

**Root Entity**: `AuditLogEntry` (ENT-CR-002) — Immutable, append-only

```python
class AuditLogEntry(AggregateRoot[AuditLogEntryId]):
    id: AuditLogEntryId
    tenant_id: TenantId
    correlation_id: CorrelationId  # VO-CR-003
    causation_id: Optional[CausationId]  # VO-CR-004
    event_type: str  # Namespaced: "nxa.assets.AssetStateChanged"
    actor: ActorInfo  # VO-CR-005 (user_id, type, ip, user_agent)
    resource: ResourceInfo  # VO-CR-006 (type, id, tenant_id)
    action: str  # CREATE, READ, UPDATE, DELETE, STATE_CHANGE, etc.
    outcome: Outcome  # SUCCESS, FAILURE, DENIED
    changes: Optional[ChangeSet]  # VO-CR-007 (before/after)
    metadata: Dict[str, str]
    timestamp: DateTime  # RFC3339 with nanoseconds
    merkle_proof: Optional[MerkleProof]  # VO-CR-008 (for verification)
```

### 9.3 Aggregate: BackgroundJob (AGG-CR-003)

**Root Entity**: `BackgroundJob` (ENT-CR-003)

```python
class BackgroundJob(AggregateRoot[BackgroundJobId]):
    id: BackgroundJobId
    tenant_id: Optional[TenantId]  # None = platform job
    job_type: str  # e.g., "sync_lansweeper", "generate_invoice", "run_assessment"
    payload: Dict[str, Any]
    priority: JobPriority  # LOW, NORMAL, HIGH, CRITICAL
    status: JobStatus  # PENDING, SCHEDULED, RUNNING, COMPLETED, FAILED, DEAD_LETTER
    scheduled_at: Optional[DateTime]
    started_at: Optional[DateTime]
    completed_at: Optional[DateTime]
    attempts: int
    max_attempts: int
    last_error: Optional[str]
    result: Optional[Dict[str, Any]]
    idempotency_key: IdempotencyKey
    created_at: DateTime
    updated_at: DateTime
```

### 9.4 Value Objects

| VO ID | Name | Description |
|-------|------|-------------|
| **VO-CR-001** | `TargetingRule` | attribute (tenant_id, user_id, custom), operator (IN, NOT_IN, MATCHES), values |
| **VO-CR-002** | `FlagVariant` | key, value, weight (for gradual rollout) |
| **VO-CR-003** | `CorrelationId` | UUID v7 (request tracing) |
| **VO-CR-004** | `CausationId` | UUID v7 (event that caused this) |
| **VO-CR-005** | `ActorInfo` | user_id, actor_type (USER, SYSTEM, SERVICE), ip, user_agent, impersonator_id |
| **VO-CR-006** | `ResourceInfo` | resource_type, resource_id, tenant_id |
| **VO-CR-007** | `ChangeSet` | before (Dict), after (Dict), diff_summary |
| **VO-CR-008** | `MerkleProof` | leaf_hash, path (List[Hash]), root_hash |

### 9.5 Domain Events (EVT-CR-###)

| Event ID | Name | Payload | Trigger |
|----------|------|---------|---------|
| **EVT-CR-001** | `FeatureFlagCreated` | `flag_id, key, flag_type, is_active` | Flag creation |
| **EVT-CR-002** | `FeatureFlagUpdated` | `flag_id, changed_fields, updated_by` | Flag change |
| **EVT-CR-003** | `AuditLogEntryCreated` | `entry_id, tenant_id, event_type, actor_type, outcome` | Every auditable action |
| **EVT-CR-004** | `BackgroundJobScheduled` | `job_id, job_type, tenant_id, scheduled_at, priority` | Job queued |
| **EVT-CR-005** | `BackgroundJobCompleted` | `job_id, job_type, duration_ms, result_summary` | Job done |
| **EVT-CR-006** | `BackgroundJobFailed` | `job_id, job_type, error, attempts, will_retry` | Job failed |

---

## 10. Cross-Context Integration Events

Integration events (IEV-###) are published to the event bus (Kafka) for cross-context communication. They are **immutable**, **versioned**, and **backward-compatible**.

| IEV ID | Event Name | Publisher | Consumers | Payload Schema (Avro/JSON Schema) |
|--------|------------|-----------|-----------|-----------------------------------|
| **IEV-001** | `TenantProvisioned` | nxa_tenants | nxa_accounts, nxa_assets, nxa_billing, nxa_notifications, nxa_audit | `{tenant_id, name, slug, region, plan, admin_user_id}` |
| **IEV-002** | `TenantSuspended` | nxa_tenants | nxa_accounts, nxa_assets, nxa_billing, nxa_notifications | `{tenant_id, reason, suspended_at}` |
| **IEV-003** | `TenantOffboarded` | nxa_tenants | All | `{tenant_id, certificate_id, offboarded_at}` |
| **IEV-004** | `UserCreated` | nxa_accounts | nxa_tenants, nxa_notifications | `{user_id, email, tenant_id, roles}` |
| **IEV-005** | `UserAuthenticated` | nxa_accounts | nxa_audit, nxa_notifications | `{user_id, tenant_id, method, mfa_used, ip, timestamp}` |
| **IEV-006** | `AssetDiscovered` | nxa_lansweeper | nxa_assets, nxa_audit | `{asset_id, tenant_id, source, external_id, raw_hash, discovered_at}` |
| **IEV-007** | `AssetReconciled` | nxa_assets | nxa_audit, nxa_billing, nxa_notifications | `{asset_id, tenant_id, merge_decisions, confidence, state}` |
| **IEV-008** | `AssetStateChanged` | nxa_assets | nxa_billing, nxa_notifications, nxa_audit, nxa_lansweeper | `{asset_id, tenant_id, from_state, to_state, reason, changed_by, timestamp}` |
| **IEV-009** | `CiRelationshipChanged` | nxa_assets | nxa_audit | `{relationship_id, source_id, target_id, type, change_type, changed_by}` |
| **IEV-010** | `MaintenanceCompleted` | nxa_assets | nxa_notifications, nxa_audit, nxa_billing | `{window_id, asset_id, tenant_id, outcome, completed_at}` |
| **IEV-011** | `SubscriptionCreated` | nxa_billing | nxa_tenants, nxa_notifications, nxa_audit | `{subscription_id, tenant_id, plan_id, billing_cycle, trial_end}` |
| **IEV-012** | `InvoiceGenerated` | nxa_billing | nxa_tenants, nxa_notifications, nxa_audit | `{invoice_id, tenant_id, subscription_id, total, due_date, period}` |
| **IEV-013** | `InvoicePaid` | nxa_billing | nxa_tenants, nxa_notifications, nxa_audit | `{invoice_id, tenant_id, amount, payment_method, paid_at}` |
| **IEV-014** | `ComplianceFindingCreated` | nxa_audit | nxa_notifications, nxa_assets, nxa_tenants | `{finding_id, tenant_id, control_id, resource, status, risk_score}` |
| **IEV-015** | `EvidencePackageGenerated` | nxa_audit | nxa_notifications, nxa_tenants | `{package_id, tenant_id, framework_id, control_id, merkle_root}` |
| **IEV-016** | `SyncJobCompleted` | nxa_lansweeper | nxa_assets, nxa_audit, nxa_notifications | `{job_id, site_id, tenant_id, stats, status}` |
| **IEV-017** | `ReconciliationConflictDetected` | nxa_lansweeper | nxa_assets, nxa_notifications | `{conflict_id, asset_id, tenant_id, field_path, source_values}` |
| **IEV-018** | `NotificationQueued` | nxa_notifications | nxa_audit | `{notification_id, tenant_id, rule_id, channel_type, recipient_count}` |

> **Event Schema Registry**: All integration events registered in `api/contracts/events/` with Avro schemas, versioned (v1, v2...), compatibility: BACKWARD.

---

## 11. Domain Services & Specifications

Domain services encapsulate logic that doesn't naturally fit in a single aggregate. Specifications encapsulate business rules for reuse.

### 11.1 Domain Services

| Service ID | Name | Context | Responsibility |
|------------|------|---------|----------------|
| **DSV-001** | `AssetReconciliationService` | nxa_assets | Multi-source reconciliation, Field Ownership Matrix application, conflict detection |
| **DSV-002** | `TenantProvisioningService` | nxa_tenants | Schema creation, RLS policy installation, default data seeding, branding setup |
| **DSV-003** | `ComplianceAssessmentService` | nxa_audit | Policy evaluation orchestration, evidence collection, finding generation |
| **DSV-004** | `CostAllocationService` | nxa_billing | Allocation rule execution, chargeback calculation, report generation |
| **DSV-005** | `NotificationDeliveryService` | nxa_notifications | Channel routing, retry logic, dead-letter handling, delivery tracking |
| **DSV-006** | `SyncOrchestrationService` | nxa_lansweeper | Job scheduling, pagination, rate limiting, checkpoint/resume |
| **DSV-007** | `ImpersonationService` | nxa_accounts | Approval workflow, session creation, audit trail, auto-expiry |
| **DSV-008** | `AssetLifecycleService` | nxa_assets | State machine enforcement, transition validation, side effects |

### 11.2 Specifications (Reusable Business Rules)

| Spec ID | Name | Context | Rule |
|---------|------|---------|------|
| **SPEC-001** | `AssetAuthorizationPolicy` | nxa_assets | Asset can be authorized iff: tenant quota not exceeded, asset type allowed, policy eval passes |
| **SPEC-002** | `TenantQuotaPolicy` | nxa_tenants | Tenant cannot exceed plan limits on assets, API calls, storage, seats |
| **SPEC-003** | `FieldOwnershipPolicy` | nxa_lansweeper | Field value determined by source priority; lock prevents overwrite |
| **SPEC-004** | `CiRelationshipAcyclicPolicy` | nxa_assets | DEPENDS_ON graph must not contain cycles |
| **SPEC-005** | `InvoiceIdempotencyPolicy` | nxa_billing | Invoice generation idempotent per tenant per period |
| **SPEC-006** | `DataResidencyPolicy` | nxa_tenants | Tenant resources must remain in provisioned region |
| **SPEC-007** | `EncryptionKeyPolicy` | nxa_tenants | Per-tenant DEK rotated annually; wrapped by platform KEK in HSM |
| **SPEC-008** | `RetentionPolicy` | nxa_audit | Data retained per classification; legal hold suspends deletion |

---

## 12. Invariants & Business Rules (Domain Level)

These invariants are enforced **within the domain layer** (no infrastructure dependencies).

### 12.1 Asset Invariants (nxa_assets)

| Invariant ID | Rule | Enforcement Point |
|--------------|------|-------------------|
| **INV-AS-001** | Asset must have exactly one current owner when in ACTIVE state | `Asset.activate()` |
| **INV-AS-002** | Asset disposal requires verified NIST 800-88 sanitization certificate | `Asset.dispose()` |
| **INV-AS-003** | CI relationship graph (DEPENDS_ON) must be acyclic | `Asset.add_relationship()`, `CIRelationship.validate_acyclic()` |
| **INV-AS-004** | Asset state transitions must follow defined state machine | All state transition methods |
| **INV-AS-005** | Field ownership matrix must be respected on all updates | `Asset.update_hardware_specs()`, `update_cloud_identity()`, `update_ownership()` |
| **INV-AS-006** | Confidence score must be recalculated on every reconciliation | `AssetReconciliationService.reconcile()` |
| **INV-AS-007** | Software installations must reference valid license entitlements for compliance | `Asset.add_software()`, `SoftwareLicenseReconciliationService` |

### 12.2 Tenant Invariants (nxa_tenants)

| Invariant ID | Rule | Enforcement Point |
|--------------|------|-------------------|
| **INV-TN-001** | Tenant slug must be globally unique, DNS-safe, 3-63 chars | `Tenant` factory |
| **INV-TN-002** | Tenant offboarding requires 30-day grace period unless legal hold | `Tenant.initiate_offboarding()` |
| **INV-TN-003** | Tenant data residency locked at provisioning; migration requires workflow | `Tenant.set_region()` (disabled after provision) |
| **INV-TN-004** | Quota allocations must not exceed plan maximums | `Tenant.update_quotas()` |
| **INV-TN-005** | Parent-child hierarchy depth ≤ 3 levels | `Tenant.set_parent()` |

### 12.3 Identity Invariants (nxa_accounts)

| Invariant ID | Rule | Enforcement Point |
|--------------|------|-------------------|
| **INV-AC-001** | User must have at least one MFA method if tenant policy requires MFA | `User.disable_mfa()` |
| **INV-AC-002** | Custom roles cannot include platform-only permissions | `Role.add_permission()` |
| **INV-AC-003** | Role inheritance depth ≤ 5; no circular references | `Role.set_parent()` |
| **INV-AC-004** | API keys hashed with bcrypt at rest; never logged | `User.add_api_key()` |
| **INV-AC-005** | Impersonation sessions max 60 minutes; require approval | `ImpersonationService.start()` |

### 12.4 Billing Invariants (nxa_billing)

| Invariant ID | Rule | Enforcement Point |
|--------------|------|-------------------|
| **INV-BL-001** | Invoice generation idempotent per tenant per billing period | `Invoice` factory (idempotency key) |
| **INV-BL-002** | Allocation rule percentages must sum to 100% per dimension per period | `AllocationRule.validate()` |
| **INV-BL-003** | Subscription quantity cannot exceed plan limits | `Subscription.update_quantity()` |
| **INV-BL-004** | Contract renewal alerts at 90/60/30 days; escalation path enforced | `Contract.check_renewal()` scheduler |

### 12.5 Audit Invariants (nxa_audit)

| Invariant ID | Rule | Enforcement Point |
|--------------|------|-------------------|
| **INV-AU-001** | Evidence packages must be cryptographically signed (JWS RS256) | `EvidencePackage` factory |
| **INV-AU-002** | Merkle tree root published hourly to transparency log | `EvidencePackage` background job |
| **INV-AU-003** | Audit log entries immutable; append-only | `AuditLogEntry` factory (no update/delete methods) |
| **INV-AU-004** | Critical findings (risk_score ≥ 9.0) must create incident within 15 min | `ComplianceAssessmentService` event handler |

---

## 13. Diagrams

| Diagram ID | Title | Type | Location | Description |
|------------|-------|------|----------|-------------|
| **DIA-027** | Bounded Context Map | Context Map | Section 1.1 | Strategic DDD context relationships |
| **DIA-028** | User Aggregate | Class Diagram | Section 2.1 | User, ApiKey, Session, MfaMethod |
| **DIA-029** | Role Aggregate | Class Diagram | Section 2.2 | Role, Permission, inheritance |
| **DIA-030** | Tenant Aggregate | Class Diagram | Section 3.1 | Tenant, BrandingSettings, QuotaLimits |
| **DIA-031** | Asset Aggregate | Class Diagram | Section 4.1 | Asset, SoftwareInstallation, HardwareSpecs |
| **DIA-032** | Asset State Machine | State Diagram | Section 4.1 | 6 states, 8 transitions |
| **DIA-033** | CI Relationship Graph | Graph | Section 4.2 | DEPENDS_ON, HOSTS, CONNECTS_TO |
| **DIA-034** | LansweeperSite Aggregate | Class Diagram | Section 5.1 | Site, AuthConfig, FieldOwnershipRules |
| **DIA-035** | Field Ownership Matrix | Matrix | Section 5.3 | Field → Source Priority |
| **DIA-036** | Subscription Aggregate | Class Diagram | Section 6.1 | Subscription, Plan, Invoice |
| **DIA-037** | Allocation Rule Model | Class Diagram | Section 6.3 | Rule, Targets, Formula |
| **DIA-038** | Notification Delivery Flow | Sequence | Section 7.3 | Rule → Queue → Channel → Delivery |
| **DIA-039** | Compliance Assessment Flow | Flowchart | Section 8.3 | Framework → Controls → Policies → Findings |
| **DIA-040** | Evidence Package Structure | Component | Section 8.4 | Items, Merkle Tree, JWS, Custody |
| **DIA-041** | Risk Register Model | Class Diagram | Section 8.5 | Risk, Treatment, Mitigation |
| **DIA-042** | Feature Flag Evaluation | Sequence | Section 9.1 | Context → Rules → Variant |
| **DIA-043** | Audit Log Architecture | Component | Section 9.2 | Entry → Kafka → Loki/S3 → Merkle |
| **DIA-044** | Integration Event Bus | Component | Section 10 | Publishers → Kafka → Consumers |
| **DIA-045** | Domain Service Dependencies | Dependency Graph | Section 11 | Services → Aggregates → Repositories |

---

## 14. Tables

| Table ID | Title | Location | Row Count |
|----------|-------|----------|-----------|
| **TBL-064** | Bounded Context Responsibilities | Section 1.2 | 8 |
| **TBL-065** | Context Relationship Patterns | Section 1.1 | 6 |
| **TBL-066** | User Aggregate Entities & VOs | Section 2.1 | 10 |
| **TBL-067** | Role Aggregate Entities & VOs | Section 2.2 | 4 |
| **TBL-068** | nxa_accounts Domain Events | Section 2.3 | 12 |
| **TBL-069** | Tenant Aggregate VOs | Section 3.1 | 7 |
| **TBL-070** | TenantUser Aggregate | Section 3.2 | 1 |
| **TBL-071** | nxa_tenants Domain Events | Section 3.3 | 12 |
| **TBL-072** | Asset Aggregate Entities & VOs | Section 4.1 | 20 |
| **TBL-073** | CIRelationship Aggregate | Section 4.2 | 2 |
| **TBL-074** | MaintenanceWindow Aggregate | Section 4.3 | 1 |
| **TBL-075** | nxa_assets Domain Events | Section 4.4 | 15 |
| **TBL-076** | LansweeperSite Aggregate VOs | Section 5.1 | 8 |
| **TBL-077** | SyncJob Aggregate | Section 5.2 | 1 |
| **TBL-078** | ReconciliationRule Aggregate | Section 5.3 | 1 |
| **TBL-079** | nxa_lansweeper VOs | Section 5.4 | 8 |
| **TBL-080** | nxa_lansweeper Domain Events | Section 5.5 | 9 |
| **TBL-081** | Subscription Aggregate VOs | Section 6.1 | 9 |
| **TBL-082** | Invoice Aggregate Entities | Section 6.2 | 2 |
| **TBL-083** | AllocationRule Aggregate VOs | Section 6.3 | 2 |
| **TBL-084** | Contract Aggregate VOs | Section 6.4 | 3 |
| **TBL-085** | nxa_billing VOs | Section 6.5 | 9 |
| **TBL-086** | nxa_billing Domain Events | Section 6.6 | 11 |
| **TBL-087** | NotificationChannel Aggregate VOs | Section 7.1 | 5 |
| **TBL-088** | NotificationRule Aggregate VOs | Section 7.2 | 5 |
| **TBL-089** | Notification Aggregate | Section 7.3 | 1 |
| **TBL-090** | nxa_notifications VOs | Section 7.4 | 5 |
| **TBL-091** | nxa_notifications Domain Events | Section 7.5 | 7 |
| **TBL-092** | Framework/Control Aggregates | Section 8.1-8.2 | 2 |
| **TBL-093** | Assessment Aggregate Entities | Section 8.3 | 2 |
| **TBL-094** | EvidencePackage Aggregate | Section 8.4 | 1 |
| **TBL-095** | Risk Aggregate VOs | Section 8.5 | 5 |
| **TBL-096** | nxa_audit VOs | Section 8.6 | 9 |
| **TBL-097** | nxa_audit Domain Events | Section 8.7 | 9 |
| **TBL-098** | FeatureFlag Aggregate VOs | Section 9.1 | 4 |
| **TBL-099** | AuditLogEntry Aggregate VOs | Section 9.2 | 8 |
| **TBL-100** | BackgroundJob Aggregate | Section 9.3 | 1 |
| **TBL-101** | nxa_core VOs | Section 9.4 | 8 |
| **TBL-102** | nxa_core Domain Events | Section 9.5 | 6 |
| **TBL-103** | Integration Events (IEV) | Section 10 | 18 |
| **TBL-104** | Domain Services | Section 11.1 | 8 |
| **TBL-105** | Specifications | Section 11.2 | 8 |
| **TBL-106** | Asset Invariants | Section 12.1 | 7 |
| **TBL-107** | Tenant Invariants | Section 12.2 | 5 |
| **TBL-108** | Identity Invariants | Section 12.3 | 5 |
| **TBL-109** | Billing Invariants | Section 12.4 | 4 |
| **TBL-110** | Audit Invariants | Section 12.5 | 4 |
| **TBL-111** | Diagram Index | Section 13 | 19 |
| **TBL-112** | Table Index | Section 14 | 49 |

---

---

**End of DOC-04 — NexusAsset Cloud Domain Model**

*Document Control: Version 1.0.0 | Classification: Internal — Confidential | Next Review: 2026-12-09*

*For questions or updates: docs@nexusasset.cloud | Architecture Review Board: arch-review@nexusasset.cloud*