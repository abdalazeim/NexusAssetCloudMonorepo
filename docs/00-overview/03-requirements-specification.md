╔════════════════════════════════════════════════════════════════════════════════════════════════╗
║ Document ID: DOC-03                                                                            ║
║ Title: Requirements Specification — Functional & Non-Functional Requirements                   ║
║ Version: 1.0.0                                                                                 ║
║ Status: ✓ Approved                                                                             ║
║ Classification: Internal — Confidential                                                        ║
║ Authors: NexusAsset Cloud Architecture Team                                                    ║
║ Reviewers: Product Management, Engineering, Security, QA, Compliance                           ║
║ Approved By: CTO / VP Engineering / CISO                                                       ║
║ Date: 2026-11-09                                                                               ║
║ Supersedes: —                                                                                  ║
║ Related: DOC-00, DOC-01, DOC-02, DOC-04, DOC-05, DOC-06, DOC-07, DOC-08, DOC-10, DOC-13      ║
║ Traceability: REQ-FN-### ← CAP-###/UC-###, REQ-NFR-### ← CAP-###/SLO-###, TC-### ← REQ-###    ║
╚════════════════════════════════════════════════════════════════════════════════════════════════╝

# ═══════════════════════════════════════════════════════════════
# NexusAsset Cloud — Requirements Specification (DOC-03)
# ═══════════════════════════════════════════════════════════════

> **Executive Summary** — This document specifies all functional (REQ-FN-###) and non-functional (REQ-NFR-###) requirements for NexusAsset Cloud. Each requirement is traced to its originating business capability (CAP-### from DOC-02), use case (UC-### from DOC-02), and forward-traced to domain model elements (DOC-04), application architecture (DOC-05), API contracts (DOC-07), and test cases (DOC-13). Requirements follow the **SMART** criteria (Specific, Measurable, Achievable, Relevant, Testable) and use the **EARS** notation (Easy Approach to Requirements Syntax) for unambiguous specification.

---

## Table of Contents

1. [Requirements Conventions & Notation](#1-requirements-conventions--notation)
2. [Functional Requirements — Identity & Access (nxa_accounts)](#2-functional-requirements--identity--access-nxa_accounts)
3. [Functional Requirements — Tenant Management (nxa_tenants)](#3-functional-requirements--tenant-management-nxa_tenants)
4. [Functional Requirements — Asset Core (nxa_assets)](#4-functional-requirements--asset-core-nxa_assets)
5. [Functional Requirements — Lansweeper Integration (nxa_lansweeper)](#5-functional-requirements--lansweeper-integration-nxa_lansweeper)
6. [Functional Requirements — Billing & Contracts (nxa_billing)](#6-functional-requirements--billing--contracts-nxa_billing)
7. [Functional Requirements — Notifications (nxa_notifications)](#7-functional-requirements--notifications-nxa_notifications)
8. [Functional Requirements — Audit & Compliance (nxa_audit)](#8-functional-requirements--audit--compliance-nxa_audit)
9. [Functional Requirements — Core Platform (nxa_core)](#9-functional-requirements--core-platform-nxa_core)
10. [Non-Functional Requirements](#10-non-functional-requirements)
11. [Security Requirements](#11-security-requirements)
12. [Multi-Tenancy & Isolation Requirements](#12-multi-tenancy--isolation-requirements)
13. [Performance & Scalability Requirements](#13-performance--scalability-requirements)
14. [Observability & Operability Requirements](#14-observability--operability-requirements)
15. [Data & Compliance Requirements](#15-data--compliance-requirements)
16. [Traceability Matrices](#16-traceability-matrices)
17. [Diagrams](#17-diagrams)
18. [Tables](#18-tables)

---

## 1. Requirements Conventions & Notation

### 1.1 Identifier Scheme

| Prefix | Category | Example | Description |
|--------|----------|---------|-------------|
| **REQ-FN** | Functional Requirement | REQ-FN-101 | User-facing capability, traced to UC-### |
| **REQ-NFR** | Non-Functional Requirement | REQ-NFR-001 | Quality attribute, traced to SLO/Capability |
| **REQ-SEC** | Security Requirement | REQ-SEC-001 | Security-specific, subset of NFR |
| **REQ-MT** | Multi-Tenancy Requirement | REQ-MT-001 | Isolation, data segregation |
| **REQ-PF** | Performance Requirement | REQ-PF-001 | Latency, throughput, scalability |
| **REQ-OB** | Observability Requirement | REQ-OB-001 | Metrics, logs, traces, alerting |
| **REQ-DC** | Data/Compliance Requirement | REQ-DC-001 | Retention, residency, privacy |

### 1.2 EARS Notation

Requirements use **EARS** (Easy Approach to Requirements Syntax) for clarity:

| Pattern | Template | Example |
|---------|----------|---------|
| **Ubiquitous** | The system shall \<action\> | The system shall encrypt all data at rest using AES-256 |
| **Event-Driven** | When \<trigger\>, the system shall \<action\> | When AssetDiscovered event received, the system shall normalize payload within 500ms |
| **State-Driven** | While \<condition\>, the system shall \<action\> | While tenant is suspended, the system shall reject all write operations |
| **Optional Feature** | Where \<feature\>, the system shall \<action\> | Where MFA is enabled, the system shall require second factor |
| **Unwanted Behavior** | If \<condition\>, then the system shall \<action\> | If reconciliation confidence < 0.85, then the system shall queue for human review |

### 1.3 Requirement Attributes

Each requirement includes:
- **ID**: Stable identifier (e.g., REQ-FN-101)
- **Title**: Concise name
- **Statement**: EARS-formatted requirement text
- **Priority**: P0 (Must), P1 (Should), P2 (Could), P3 (Won't this release)
- **Source**: CAP-### / UC-### from DOC-02
- **Verification**: Test case IDs (TC-### from DOC-13)
- **Dependencies**: Other requirement IDs
- **Assumptions**: ASM-### from DOC-02

---

## 2. Functional Requirements — Identity & Access (nxa_accounts)

### 2.1 Authentication (UC-AC-001, UC-AC-002, UC-AC-008)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-101** | Username/Password Authentication | The system shall authenticate users via username/email and password using Argon2id (memory=64MB, iterations=3, parallelism=4) | P0 | UC-AC-001 | TC-AUTH-001, TC-AUTH-002 |
| **REQ-FN-102** | Multi-Factor Authentication (TOTP) | When MFA is enabled for a user, the system shall require a valid TOTP code (RFC 6238, 30s window, 6 digits) after successful password verification | P0 | UC-AC-001, UC-AC-008 | TC-AUTH-003, TC-AUTH-004 |
| **REQ-FN-103** | Multi-Factor Authentication (WebAuthn) | Where WebAuthn is registered, the system shall support FIDO2/CTAP2 authenticators (platform + roaming) with user verification required | P0 | UC-AC-008 | TC-AUTH-005, TC-AUTH-006 |
| **REQ-FN-104** | Backup Codes | When MFA is enabled, the system shall generate 10 single-use backup codes (256-bit entropy) and display once | P1 | UC-AC-008 | TC-AUTH-007 |
| **REQ-FN-105** | SSO (SAML 2.0) | The system shall act as SP for SAML 2.0 IdP-initiated and SP-initiated flows, validating assertions, signature, audience, and NotBefore/NotOnOrAfter | P0 | UC-AC-002 | TC-AUTH-010, TC-AUTH-011 |
| **REQ-FN-106** | SSO (OIDC) | The system shall support OIDC Authorization Code Flow with PKCE, validating ID token signature (RS256), issuer, audience, nonce, and expiry | P0 | UC-AC-002 | TC-AUTH-012, TC-AUTH-013 |
| **REQ-FN-107** | Just-In-Time (JIT) Provisioning | When SSO login succeeds for unknown user, the system shall create user account, assign default tenant role, and link identity provider subject | P0 | UC-AC-002 | TC-AUTH-014 |
| **REQ-FN-108** | JWT Token Issuance | Upon successful authentication, the system shall issue: access token (JWT, RS256, 15min TTL, `sub`, `tid`, `roles`, `scopes`, `permissions`) and refresh token (opaque, 30d TTL, rotating) | P0 | UC-AC-001, UC-AC-002 | TC-AUTH-020, TC-AUTH-021 |
| **REQ-FN-109** | Token Refresh | When valid refresh token presented, the system shall issue new access+refresh pair, revoke old refresh token, and log rotation event | P0 | UC-AC-001 | TC-AUTH-022 |
| **REQ-FN-110** | Token Revocation | When logout or security event occurs, the system shall revoke refresh token and add access token JTI to denylist (Redis, TTL=access token remaining) | P0 | UC-AC-001 | TC-AUTH-023 |

### 2.2 Authorization (UC-AC-005, UC-AC-006)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-111** | RBAC Enforcement | The system shall enforce role-based access control: each API endpoint declares required permissions; access granted iff user's effective permissions (role + direct + group) include required | P0 | UC-AC-005 | TC-AUTHZ-001..010 |
| **REQ-FN-112** | Custom Role Creation | Tenant admins shall create custom roles by selecting from permission catalog; system validates no platform-only permissions included | P0 | UC-AC-005 | TC-AUTHZ-011, TC-AUTHZ-012 |
| **REQ-FN-113** | Permission Boundaries | Where permission boundary assigned to role, the system shall intersect role permissions with boundary before grant | P1 | UC-AC-005 | TC-AUTHZ-013 |
| **REQ-FN-114** | ABAC Policy Evaluation | When ABAC enabled for resource, the system shall evaluate OPA policy with context: user attributes, resource tags, request metadata, time, location, risk score | P1 | UC-AC-006 | TC-AUTHZ-020..030 |
| **REQ-FN-115** | Delegated Administration | Tenant admins shall delegate role assignment to other users with scope limitation (specific roles, specific user groups) | P1 | UC-AC-005 | TC-AUTHZ-014, TC-AUTHZ-015 |

### 2.3 User & Session Management (UC-AC-003, UC-AC-004, UC-AC-007, UC-AC-009, UC-AC-010)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-116** | Profile Management | Users shall update display name, avatar, timezone, language, notification preferences via API/UI | P1 | UC-AC-003 | TC-USER-001 |
| **REQ-FN-117** | API Key Management | Users shall create/revoke/list API keys with scopes, expiry, description; keys hashed (bcrypt) at rest | P0 | UC-AC-004 | TC-USER-010..015 |
| **REQ-FN-118** | Impersonation (Audited) | Support admins shall impersonate users with: time-boxed (max 60min), requires approval workflow, full audit trail, MFA re-verification | P1 | UC-AC-007 | TC-USER-020, TC-USER-021, TC-SEC-010 |
| **REQ-FN-119** | Session Management | Users shall view active sessions (device, IP, location, last active), revoke individual or all sessions | P1 | UC-AC-009 | TC-USER-030 |
| **REQ-FN-120** | Concurrent Session Limit | Where tenant configures max concurrent sessions, the system shall enforce limit by revoking oldest session on new login | P1 | UC-AC-009 | TC-USER-031 |
| **REQ-FN-121** | SCIM 2.0 Provisioning | The system shall expose SCIM 2.0 endpoint (`/scim/v2`) for User/Group create/read/update/delete/list with filtering and pagination | P1 | UC-AC-010 | TC-USER-040..050 |

---

## 3. Functional Requirements — Tenant Management (nxa_tenants)

### 3.1 Tenant Lifecycle (UC-TN-001, UC-TN-007, UC-TN-008)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-201** | Self-Service Tenant Creation | Prospects shall register tenant via public signup: org name, admin email, subdomain, plan; system provisions schema, RLS policies, default roles, branding defaults within 60 seconds | P0 | UC-TN-001 | TC-TENANT-001..005 |
| **REQ-FN-202** | Tenant Provisioning Idempotency | Tenant provisioning shall be idempotent: re-running with same external ID returns existing tenant without side effects | P0 | UC-TN-001 | TC-TENANT-006 |
| **REQ-FN-203** | Tenant Suspension | Platform admins shall suspend tenant: system sets `suspended_at`, revokes all sessions, rejects write API calls (403), preserves data, allows read-only for admins | P1 | UC-TN-007 | TC-TENANT-010, TC-TENANT-011 |
| **REQ-FN-204** | Tenant Reactivation | Platform admins shall reactivate suspended tenant: clears `suspended_at`, restores write access, sends notification | P1 | UC-TN-007 | TC-TENANT-012 |
| **REQ-FN-205** | Tenant Offboarding (Secure Delete) | Platform admins shall offboard tenant: verify no legal hold → export all data (signed package) → cryptographic erase (per-tenant DEK destruction) → delete schema → issue destruction certificate | P0 | UC-TN-008 | TC-TENANT-020..025, TC-SEC-020 |
| **REQ-FN-206** | Grace Period Enforcement | When offboarding initiated, system shall enforce 30-day grace period (configurable) unless legal hold flag set; tenant admin can cancel during grace period | P0 | UC-TN-008 | TC-TENANT-026 |

### 3.2 Tenant Configuration (UC-TN-002, UC-TN-003, UC-TN-004, UC-TN-005, UC-TN-009, UC-TN-010)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-207** | Tenant Settings Management | Tenant admins shall configure: custom fields, workflows, retention policies, integration credentials, notification templates, branding | P0 | UC-TN-002 | TC-TENANT-030..040 |
| **REQ-FN-208** | User/Group Management | Tenant admins shall invite users (email + role), bulk import (CSV/SCIM), assign groups, manage group membership | P0 | UC-TN-003 | TC-TENANT-050..060 |
| **REQ-FN-209** | Usage & Quota Dashboard | Tenant admins shall view real-time: asset count, API calls/day, storage GB, scan frequency, seat utilization vs plan limits | P0 | UC-TN-004 | TC-TENANT-070 |
| **REQ-FN-210** | Quota Increase Request | Tenant admins shall request quota increase: workflow routes to platform admin → approval → auto-provision → notification | P1 | UC-TN-005 | TC-TENANT-071, TC-TENANT-072 |
| **REQ-FN-211** | Data Export (GDPR/DSAR) | Tenant admins shall export all tenant data in JSON/CSV with signed manifest, delivered via secure download (expiring link, 7 days) | P0 | UC-TN-006 | TC-TENANT-080, TC-SEC-030 |
| **REQ-FN-212** | Custom Domain & TLS | Tenant admins shall configure custom domain: system verifies CNAME, provisions Let's Encrypt certificate (auto-renewal), serves on `app.nexusasset.cloud` edge | P1 | UC-TN-009 | TC-TENANT-090..095 |
| **REQ-FN-213** | Tenant Hierarchy (Parent/Child) | Platform admins shall create parent-child tenant relationships: consolidated billing, policy inheritance (opt-in), cross-tenant reporting | P2 | UC-TN-010 | TC-TENANT-100..110 |

---

## 4. Functional Requirements — Asset Core (nxa_assets)

### 4.1 Asset Registration & Management (UC-AS-001, UC-AS-002, UC-AS-005, UC-AS-008, UC-AS-009, UC-AS-012)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-301** | Manual Asset Registration | Authorized users shall register assets via API/UI: type, serial, model, manufacturer, owner, location, tags, custom fields, attachments (max 50MB each) | P0 | UC-AS-001 | TC-ASSET-001..010 |
| **REQ-FN-302** | Bulk Import | Authorized users shall import assets via CSV/Excel: system validates schema, shows preview with errors, supports rollback on failure, provides template download | P0 | UC-AS-002 | TC-ASSET-011..020 |
| **REQ-FN-303** | Asset 360° View | Authorized users shall view asset details: properties, relationships (graph), history (audit), compliance findings, financials, linked tickets, scan sources | P0 | UC-AS-003 | TC-ASSET-030 |
| **REQ-FN-304** | Search & Filter | Users shall search assets: full-text (name, serial, tags), faceted (type, state, owner, location, source), saved queries, export (CSV/JSON), API with cursor pagination | P0 | UC-AS-004 | TC-ASSET-040..050 |
| **REQ-FN-305** | Asset Assignment | Authorized users shall assign/reassign assets to users/groups/locations: custody transfer workflow with optional signature capture, full history | P0 | UC-AS-005 | TC-ASSET-060..070 |
| **REQ-FN-306** | Relocation Tracking | Authorized users shall record physical/logical moves: from/to location, rack/U position, geo-coordinates, reason, responsible party, timestamp | P1 | UC-AS-008 | TC-ASSET-080 |
| **REQ-FN-307** | Retirement & Disposal Workflow | Authorized users shall retire assets: workflow enforces sanitization verification (NIST 800-88), disposal certificate upload, financial write-off, CMDB cleanup | P0 | UC-AS-009 | TC-ASSET-090..100, TC-SEC-040 |
| **REQ-FN-308** | Custom Fields & Schemas | Tenant admins shall define custom fields: text, number, date, datetime, enum, boolean, reference (to asset/user), with validation rules (regex, min/max, required) | P1 | UC-AS-012 | TC-ASSET-110..120 |

### 4.2 Maintenance Management (UC-AS-006, UC-AS-007)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-309** | Maintenance Scheduling | Authorized users shall schedule maintenance: preventive (recurring cron) or corrective (ad-hoc), SLA target, assigned technician/team, required parts, downtime window | P1 | UC-AS-006 | TC-MAINT-001..010 |
| **REQ-FN-310** | Maintenance Execution | Technicians shall record outcome: status (completed/partial/failed), actual start/end, parts consumed, labor hours, notes, photos, next service date recommendation | P1 | UC-AS-007 | TC-MAINT-011..020 |
| **REQ-FN-311** | SLA Tracking & Alerts | System shall track maintenance SLA: alert at 50%, 80%, 100% of target; escalate on breach per tenant config | P1 | UC-AS-006 | TC-MAINT-021..030 |

### 4.3 CMDB & Relationships (UC-AS-010, UC-AS-011)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-312** | CI Relationship Management | CMDB admins shall create/update/delete relationships: `depends_on`, `hosts`, `connects_to`, `contains`, `runs_on` with visual topology editor | P0 | UC-AS-010 | TC-CMDB-001..015 |
| **REQ-FN-313** | Relationship Validation | System shall reject circular dependencies in `depends_on` graph; validate cardinality constraints per relationship type | P0 | UC-AS-010 | TC-CMDB-016..020 |
| **REQ-FN-314** | Impact Analysis | Given a CI, system shall compute upstream/downstream impact set (transitive closure) for change risk assessment | P0 | UC-AS-010 | TC-CMDB-021..030 |
| **REQ-FN-315** | Asset Comparison & Diff | Users shall compare two asset versions (scan vs manual, two scans): side-by-side diff with source attribution, field-level change history | P1 | UC-AS-011 | TC-ASSET-130..140 |

---

## 5. Functional Requirements — Lansweeper Integration (nxa_lansweeper)

### 5.1 Connection & Configuration (UC-LS-001, UC-LS-010)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-401** | Lansweeper Site Configuration | Platform admins shall configure Lansweeper sites: site name, GraphQL endpoint URL, authentication (PAT/OAuth2 client credentials), sync schedule (cron), asset type filters | P0 | UC-LS-001 | TC-LS-001..010 |
| **REQ-FN-402** | Credential Vault Integration | Lansweeper credentials shall be stored in HashiCorp Vault / CyberArk; system rotates per policy, tests connectivity on save | P0 | UC-LS-010 | TC-LS-011, TC-SEC-050 |
| **REQ-FN-403** | Connection Health Check | System shall test Lansweeper connectivity on config save and every 5 min; alert on 3 consecutive failures | P0 | UC-LS-001, UC-LS-010 | TC-LS-012 |

### 5.2 Synchronization (UC-LS-002, UC-LS-003, UC-LS-006, UC-LS-007)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-404** | Manual Sync Trigger | Authorized users shall trigger on-demand sync: full (all assets) or incremental (since last successful sync) for specific site or all sites | P0 | UC-LS-002 | TC-LS-020..030 |
| **REQ-FN-405** | Scheduled Sync | System shall execute sync per site schedule (cron): incremental by default, full weekly; respects rate limits, handles pagination | P0 | UC-LS-001 | TC-LS-031..040 |
| **REQ-FN-406** | Sync Status & History | Users shall view: current sync progress (assets processed, rate), last 50 sync runs (status, duration, assets created/updated/deleted, errors) | P0 | UC-LS-003 | TC-LS-050 |
| **REQ-FN-407** | Webhook Registration | Platform admins shall register Lansweeper webhooks: asset.created, asset.updated, asset.deleted; system validates signature, configures retry (exponential backoff, max 5), dead-letter queue | P1 | UC-LS-006 | TC-LS-060..070 |
| **REQ-FN-408** | Delta Sync & Change Detection | On webhook or scheduled incremental, system shall detect changes via `lastChanged` timestamp; batch upsert (1000 assets/batch), emit `AssetDiscovered` per asset | P0 | UC-LS-007 | TC-LS-080..090 |
| **REQ-FN-409** | Sync Idempotency | Sync operations shall be idempotent: re-processing same Lansweeper asset payload produces same CMDB state (deterministic reconciliation) | P0 | UC-LS-002, UC-LS-007 | TC-LS-100, TC-LS-101 |

### 5.3 Field Ownership & Conflict Resolution (UC-LS-004, UC-LS-005)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-410** | Field Ownership Matrix Configuration | Data stewards shall configure per-field ownership: source priority (Lansweeper > Cloud > Manual), conflict resolution (source-wins, last-write-wins, merge), lock (prevent overwrite) | P0 | UC-LS-004 | TC-LS-110..120 |
| **REQ-FN-411** | Conflict Detection & Queue | When reconciliation detects conflict (different values for same field from different sources), system shall queue for steward review with side-by-side comparison | P0 | UC-LS-005 | TC-LS-121..130 |
| **REQ-FN-412** | Conflict Resolution Actions | Stewards shall resolve: accept source A, accept source B, merge values, mark reviewed (no change); decision logged with actor, timestamp, reasoning | P0 | UC-LS-005 | TC-LS-131..140 |

### 5.4 Software & Vulnerability (UC-LS-008, UC-LS-009)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-413** | Software License Reconciliation | System shall match discovered software (name, version, edition) to license entitlements (SKU, metrics: per-device, per-core, per-user); calculate compliance position (compliant, under-licensed, over-licensed) | P1 | UC-LS-008 | TC-LS-150..160 |
| **REQ-FN-414** | CVE Correlation | System shall correlate discovered software versions with NVD CVE database (daily sync); compute risk score (CVSS × exploitability × asset criticality); generate findings | P1 | UC-LS-009 | TC-LS-170..180, TC-SEC-060 |

---

## 6. Functional Requirements — Billing & Contracts (nxa_billing)

### 6.1 Subscription & Plan Management (UC-BL-001, UC-BL-002)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-501** | Plan Catalog Management | Product admins shall manage plans: name, tier (Free/Pro/Enterprise), billing cycle (monthly/annual), price per unit (asset/user/seat), feature flags, limits (assets, API calls, scans, storage, seats) | P0 | UC-BL-001 | TC-BILL-001..010 |
| **REQ-FN-502** | Subscription Purchase/Upgrade | Tenant admins shall purchase/upgrade via self-service: Stripe checkout, proration calculation, immediate provisioning for upgrades, scheduled downgrade at period end, cancellation with grace period | P0 | UC-BL-002 | TC-BILL-011..020 |
| **REQ-FN-503** | Trial Management | System shall support free trials: configurable duration, feature access, automatic conversion or expiration, trial-extension workflow | P1 | UC-BL-001 | TC-BILL-021..030 |

### 6.2 Invoicing & Payments (UC-BL-003, UC-BL-009)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-504** | Invoice Generation | System shall generate invoices per billing cycle: recurring (subscription) + usage (overages), line items with description/quantity/unit price, tax calculation (VAT/GST per jurisdiction), PDF + JSON | P0 | UC-BL-003 | TC-BILL-040..050 |
| **REQ-FN-505** | Invoice Idempotency | Invoice generation shall be idempotent per tenant per period: re-run produces identical invoice (same ID, line items, totals) | P0 | UC-BL-003 | TC-BILL-051 |
| **REQ-FN-506** | Payment Processing | System shall process payments via Stripe: card, ACH, SEPA, wire; handle 3D Secure, retry failed payments (dunning: 3/7/14 days), update invoice status | P0 | UC-BL-003, UC-BL-009 | TC-BILL-060..070 |
| **REQ-FN-507** | Payment Method Management | Tenant admins shall manage payment methods: add/remove/set default, validate on add, support multiple per tenant | P0 | UC-BL-009 | TC-BILL-080..090 |

### 6.3 Cost Allocation & Chargeback (UC-BL-004, UC-BL-005)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-508** | Allocation Rule Engine | Finance admins shall define allocation rules: dimensions (tenant, department, cost center, project), models (proportional by asset count, fixed, tiered, custom formula), effective dates | P0 | UC-BL-004 | TC-BILL-100..110 |
| **REQ-FN-509** | Allocation Validation | System shall validate allocation rules sum to 100% per dimension per period; reject save if not | P0 | UC-BL-004 | TC-BILL-111 |
| **REQ-FN-510** | Chargeback/Showback Reports | Finance admins shall generate reports: allocated cost per dimension, budget vs actual, variance %, trend, export (CSV/Excel/PDF), schedule recurring | P1 | UC-BL-005 | TC-BILL-120..130 |

### 6.4 Contracts & License True-Up (UC-BL-006, UC-BL-007, UC-BL-008, UC-BL-010)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-511** | Contract Repository | Procurement shall store contracts: vendor, type (SaaS, hardware, support), term, value, renewal date, obligations (SLAs, penalties), documents (PDF), approval workflow | P1 | UC-BL-006 | TC-BILL-140..150 |
| **REQ-FN-512** | Renewal Alerts | System shall alert on contract renewal: 90/60/30 days before; escalate to owner → manager → procurement | P1 | UC-BL-006 | TC-BILL-151..160 |
| **REQ-FN-513** | License True-Up Automation | System shall compare discovered installations (from Lansweeper) vs entitlements (contract repository); generate true-up order with quantity, cost, vendor SKU | P1 | UC-BL-007 | TC-BILL-170..180 |
| **REQ-FN-514** | Multi-Currency & Tax | System shall support: base currency per tenant, daily FX rates (ECB/fixer.io), tax rules per jurisdiction (VAT, GST, US sales tax), tax-inclusive/exclusive pricing | P1 | UC-BL-008 | TC-BILL-190..200 |
| **REQ-FN-515** | Financial Dashboard | CFO/Finance shall view: MRR/ARR, churn, expansion, contraction, NRR, LTV, CAC, cohort analysis, revenue recognition schedule | P2 | UC-BL-010 | TC-BILL-210..220 |

---

## 7. Functional Requirements — Notifications (nxa_notifications)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-601** | Channel Configuration | Tenant admins shall configure channels: Email (SMTP/SendGrid), SMS (Twilio), Slack, Teams, Webhook, Push (FCM/APNs), In-app; with templates (Handlebars), branding | P0 | UC-NT-001 | TC-NOTIF-001..010 |
| **REQ-FN-602** | Notification Rules | Tenant admins shall define rules: trigger event (asset state change, compliance finding, sync failure, threshold breach), conditions (JMESPath), channels, template, frequency (immediate/digest), quiet hours | P0 | UC-NT-002 | TC-NOTIF-011..020 |
| **REQ-FN-603** | User Preferences | Users shall set per-channel preferences: enable/disable, digest frequency, quiet hours, severity filter | P1 | UC-NT-003 | TC-NOTIF-030..040 |
| **REQ-FN-604** | Notification History & Delivery | Users shall view history: searchable (event, channel, status, date), delivery receipts (sent/delivered/opened/failed), resend failed | P1 | UC-NT-004 | TC-NOTIF-050..060 |
| **REQ-FN-605** | Broadcast Announcements | Platform admins shall send system-wide announcements: target (all/tenant/role), channels, scheduling, acknowledgment tracking | P1 | UC-NT-005 | TC-NOTIF-070..080 |
| **REQ-FN-606** | Delivery Guarantees | System shall guarantee at-least-once delivery: persistent queue (RabbitMQ), retry with exponential backoff (max 72h), dead-letter after max retries, idempotent consumers | P0 | UC-NT-001..005 | TC-NOTIF-090..100 |

---

## 8. Functional Requirements — Audit & Compliance (nxa_audit)

### 8.1 Framework & Policy Management (UC-AU-001, UC-AU-002, UC-AU-009)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-701** | Framework Library | Compliance admins shall manage frameworks: ISO 27001, SOC 2, GDPR, HIPAA, PCI-DSS, NIST CSF, CIS Controls, custom; each with controls, evidence requirements, testing procedures, frequency | P0 | UC-AU-001 | TC-AUDIT-001..010 |
| **REQ-FN-702** | Control-Policy Mapping | Compliance admins shall map controls to OPA policies: control ID, policy reference, evidence type, pass/fail criteria, remediation guidance | P0 | UC-AU-002 | TC-AUDIT-011..020 |
| **REQ-FN-703** | Policy-as-Code Management | Platform admins shall author/test/deploy OPA policies: Rego syntax, unit tests (opa test), versioning (Git), staged rollout (canary), drift detection | P0 | UC-AU-009 | TC-AUDIT-021..030, TC-SEC-070 |

### 8.2 Assessment & Evidence (UC-AU-003, UC-AU-004, UC-AU-010)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-704** | Compliance Assessment Engine | System shall run assessments: on-demand or scheduled; evaluate all mapped policies; collect evidence (config snapshots, logs, scan results); produce findings (PASS/FAIL/WARN) with evidence refs | P0 | UC-AU-003 | TC-AUDIT-040..050 |
| **REQ-FN-705** | Evidence Package Generation | System shall generate audit evidence packages: immutable, cryptographically signed (JWS RS256), Merkle tree inclusion proof, chain of custody, delivered via secure download | P0 | UC-AU-004 | TC-AUDIT-060..070, TC-SEC-080 |
| **REQ-FN-706** | Compliance Dashboard | Executives shall view real-time posture: framework scorecards, trend, control status, findings by severity, benchmark vs peers, board-ready PDF export | P0 | UC-AU-010 | TC-AUDIT-080..090 |

### 8.3 Risk & Incident Management (UC-AU-005, UC-AU-006, UC-AU-007, UC-AU-008)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-707** | Risk Register | Risk owners shall manage risks: ID, title, category, likelihood (1-5), impact (1-5), inherent/residual score, treatment (accept/mitigate/transfer/avoid), owner, review date, status | P1 | UC-AU-005 | TC-AUDIT-100..110 |
| **REQ-FN-708** | Incident Management | Security team shall manage incidents: timeline, evidence preservation (immutable copy), classification (P1-P4), notification workflows (GDPR 72h, customer), root cause analysis, post-mortem | P0 | UC-AU-006 | TC-AUDIT-120..140, TC-SEC-090 |
| **REQ-FN-709** | Data Classification | Data stewards shall classify data assets: labels (Public/Internal/Confidential/Restricted), handling procedures, DLP rules, retention/disposal schedule, lineage | P1 | UC-AU-007 | TC-AUDIT-150..160 |
| **REQ-FN-710** | Vendor Risk Assessment | Procurement shall assess vendors: questionnaire (SIG Lite/CAIQ), scoring, continuous monitoring (security ratings), contract compliance tracking | P2 | UC-AU-008 | TC-AUDIT-170..180 |

---

## 9. Functional Requirements — Core Platform (nxa_core)

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-FN-801** | API Gateway | Platform shall provide API gateway: routing, rate limiting (token bucket, per-tenant), authentication (JWT, API key), request/response transformation, caching, canary routing, circuit breaker | P0 | UC-CR-001 | TC-CORE-001..020 |
| **REQ-FN-802** | Developer Portal | System shall expose developer portal: OpenAPI 3.1 browsing, try-it-out, SDK generation (TypeScript, Python, Go), API key management, usage analytics | P0 | UC-CR-002 | TC-CORE-030..040 |
| **REQ-FN-803** | Feature Flags | Product/Eng shall manage flags: boolean/multivariate, targeting rules (tenant, user %, segment), gradual rollout, kill switch, audit log | P1 | UC-CR-003 | TC-CORE-050..060 |
| **REQ-FN-804** | Status Page | Public status page: component health (operational/degraded/outage), incident history, scheduled maintenance, RSS/Atom feed, embeddable badge | P1 | UC-CR-004 | TC-CORE-070..080 |
| **REQ-FN-805** | Audit Log Query | Auditors/Admins shall query audit logs: filter (tenant, user, action, resource, date range), export (CSV/JSON/CEF), retention policy enforcement | P0 | UC-CR-005 | TC-CORE-090..100 |
| **REQ-FN-806** | Background Job Platform | System shall provide job platform: scheduling (cron, calendar), priority queues, retry (exponential backoff), dead-letter, monitoring, idempotency keys | P1 | UC-CR-006 | TC-CORE-110..120 |

---

## 10. Non-Functional Requirements

### 10.1 Availability & Reliability

| ID | Title | Statement | Target | Priority | Verification |
|----|-------|-----------|--------|----------|--------------|
| **REQ-NFR-001** | Platform Availability | The platform shall achieve 99.95% monthly uptime (excluding planned maintenance windows) | 99.95% | P0 | TC-NFR-AVAIL-001 |
| **REQ-NFR-002** | API Availability | Public API shall achieve 99.99% monthly uptime | 99.99% | P0 | TC-NFR-AVAIL-002 |
| **REQ-NFR-003** | RPO (Recovery Point Objective) | Maximum data loss in disaster scenario | RPO ≤ 1 hour | P0 | TC-NFR-DR-001 |
| **REQ-NFR-004** | RTO (Recovery Time Objective) | Maximum time to restore service after disaster | RTO ≤ 4 hours | P0 | TC-NFR-DR-002 |
| **REQ-NFR-005** | Graceful Degradation | When non-critical services fail (notifications, reporting), core API (assets, auth) shall remain operational | 100% core availability | P0 | TC-NFR-AVAIL-003, TC-CHAOS-001 |
| **REQ-NFR-006** | Multi-AZ Deployment | Production shall deploy across ≥ 3 availability zones with automatic failover | 3 AZs | P0 | TC-NFR-AVAIL-004 |

### 10.2 Performance & Scalability

| ID | Title | Statement | Target | Priority | Verification |
|----|-------|-----------|--------|----------|--------------|
| **REQ-NFR-010** | API Latency (p99) | p99 latency for `/api/v1/assets/*` read endpoints | < 500 ms | P0 | TC-PERF-001 |
| **REQ-NFR-011** | API Latency (p99) Write | p99 latency for asset create/update endpoints | < 800 ms | P0 | TC-PERF-002 |
| **REQ-NFR-012** | Sync Throughput | Lansweeper delta sync shall process ≥ 10,000 assets/minute per worker | 10k assets/min | P0 | TC-PERF-010 |
| **REQ-NFR-013** | Reconciliation Throughput | Reconciliation engine shall process ≥ 5,000 assets/minute | 5k assets/min | P0 | TC-PERF-011 |
| **REQ-NFR-014** | Concurrent Users | Platform shall support ≥ 10,000 concurrent authenticated users per tenant | 10k concurrent | P1 | TC-PERF-020 |
| **REQ-NFR-015** | Tenant Scale | Platform shall support ≥ 1,000 tenants per cluster, 10M assets per tenant | 1k tenants, 10M assets | P0 | TC-PERF-030 |
| **REQ-NFR-016** | Horizontal Scaling | All stateless services shall scale horizontally via HPA (CPU > 70%, custom metrics) | Auto-scale < 3 min | P0 | TC-PERF-040 |
| **REQ-NFR-017** | Database Connection Pooling | PgBouncer pool mode transaction; max 100 connections per pod | 100 conn/pod | P0 | TC-PERF-050 |

### 10.3 Security & Privacy

| ID | Title | Statement | Target | Priority | Verification |
|----|-------|-----------|--------|----------|--------------|
| **REQ-NFR-020** | Encryption at Rest | All persistent data encrypted with AES-256; per-tenant DEK wrapped by platform KEK in HSM | AES-256 + HSM | P0 | TC-SEC-100 |
| **REQ-NFR-021** | Encryption in Transit | All network traffic TLS 1.3; mTLS between services; certificate rotation automated | TLS 1.3 + mTLS | P0 | TC-SEC-110 |
| **REQ-NFR-022** | Secret Management | All secrets in Vault; rotation every 90 days (configurable); dynamic secrets for DB | Vault + rotation | P0 | TC-SEC-120 |
| **REQ-NFR-023** | Vulnerability Scanning | Container images scanned on build; dependencies scanned daily; critical vulns block deploy | Scan on build + daily | P0 | TC-SEC-130 |
| **REQ-NFR-024** | Penetration Testing | Annual third-party pen test; quarterly automated scans; findings tracked to closure | Annual + quarterly | P0 | TC-SEC-140 |
| **REQ-NFR-025** | Data Residency | EU tenant data stored/processed only in EU regions; enforced at provisioning | EU-only for EU tenants | P0 | TC-SEC-150, TC-MT-010 |

### 10.4 Observability

| ID | Title | Statement | Target | Priority | Verification |
|----|-------|-----------|--------|----------|--------------|
| **REQ-NFR-030** | Metrics Coverage | All services export Prometheus metrics: RED (rate, errors, duration), USE (utilization, saturation, errors), business KPIs | 100% services | P0 | TC-OBS-001 |
| **REQ-NFR-031** | Structured Logging | All services emit JSON logs: trace_id, span_id, tenant_id, user_id, level, message, context; shipped to Loki | JSON + trace context | P0 | TC-OBS-010 |
| **REQ-NFR-032** | Distributed Tracing | OpenTelemetry instrumentation: 100% request trace coverage; W3C TraceContext propagation; sampling 10% (100% errors) | 100% coverage | P0 | TC-OBS-020 |
| **REQ-NFR-033** | Alerting SLA | Critical alerts (P1) notified within 1 min; warning (P2) within 5 min; via PagerDuty/Slack/Email | P1 < 1 min, P2 < 5 min | P0 | TC-OBS-030 |
| **REQ-NFR-034** | Dashboard Availability | Grafana dashboards load < 3 s; auto-refresh ≤ 30 s; embedded in portal | < 3 s load | P1 | TC-OBS-040 |
| **REQ-NFR-035** | Log Retention | Hot logs (Loki): 30 days; Cold logs (S3/Parquet): 7 years; Audit logs: immutable 7 years | 30d hot, 7yr cold | P0 | TC-OBS-050 |

---

## 11. Security Requirements

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-SEC-001** | Zero Trust Network | All service-to-service communication authenticated and authorized via mTLS + SPIFFE identities; no implicit trust | P0 | DOC-12 | TC-SEC-200 |
| **REQ-SEC-002** | RBAC/ABAC Enforcement | Every API endpoint enforces authorization; decisions logged with policy version | P0 | REQ-FN-111, REQ-FN-114 | TC-SEC-210 |
| **REQ-SEC-003** | Tenant Isolation | Cross-tenant data access prevented at database (RLS), API (tenant context), and network (namespace) layers | P0 | REQ-MT-001 | TC-SEC-220, TC-MT-001..010 |
| **REQ-SEC-004** | Input Validation | All API inputs validated via schema (Pydantic/Zod); injection prevention (SQL, NoSQL, LDAP, command) | P0 | DOC-07 | TC-SEC-230 |
| **REQ-SEC-005** | Output Encoding | All user-supplied data encoded for context (HTML, JS, SQL, URL) before rendering | P0 | DOC-07 | TC-SEC-240 |
| **REQ-SEC-006** | Security Headers | All HTTP responses include: CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy | P0 | DOC-07 | TC-SEC-250 |
| **REQ-SEC-007** | Rate Limiting | API rate limited per tenant (configurable tiers); burst protection; distributed rate limiting via Redis | P0 | REQ-FN-801 | TC-SEC-260 |
| **REQ-SEC-008** | Audit Logging | All security-relevant events logged: auth, authz, data access, config changes, privilege escalation | P0 | REQ-FN-805 | TC-SEC-270 |
| **REQ-SEC-009** | Incident Response | Automated containment for critical findings (revoke tokens, isolate tenant, block IP); playbooks for top 10 scenarios | P0 | REQ-FN-708 | TC-SEC-280, TC-CHAOS-010 |
| **REQ-SEC-010** | Supply Chain Security | SBOM (CycloneDX) generated per build; signature verification (cosign); dependency license compliance | P0 | DOC-11 | TC-SEC-290 |

---

## 12. Multi-Tenancy & Isolation Requirements

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-MT-001** | Database-Level Isolation | PostgreSQL Row-Level Security (RLS) policies on all tenant-scoped tables; `tenant_id` column mandatory; `SET LOCAL app.tenant_id` enforced by middleware | P0 | BR-TN-001 | TC-MT-001..010 |
| **REQ-MT-002** | Schema-per-Tenant Option | For high-isolation tenants: dedicated schema with own RLS; provisioned on demand | P1 | UC-TN-001 | TC-MT-011 |
| **REQ-MT-003** | Encryption-per-Tenant | Each tenant has unique Data Encryption Key (DEK) wrapped by platform KEK in HSM; DEK rotated annually | P0 | REQ-NFR-020 | TC-MT-020, TC-SEC-100 |
| **REQ-MT-004** | API Tenant Context | Every request resolved to tenant via: subdomain, header `X-Tenant-ID`, or JWT `tid` claim; middleware validates access | P0 | UC-CR-001 | TC-MT-030 |
| **REQ-MT-005** | Cross-Tenant Query Prevention | Database queries without tenant context rejected; ORM automatically scopes; raw SQL blocked by linter | P0 | BR-TN-001 | TC-MT-040 |
| **REQ-MT-006** | Tenant Resource Quotas | Per-tenant quotas enforced: assets, API calls/min, storage GB, concurrent scans, seats; hard limits with graceful degradation | P0 | UC-TN-004 | TC-MT-050 |
| **REQ-MT-007** | Network Isolation | Tenant workloads in dedicated Kubernetes namespaces; NetworkPolicies restrict inter-tenant traffic; egress controlled | P0 | DOC-06 | TC-MT-060 |
| **REQ-MT-008** | Audit Log Segregation | Audit logs partitioned by tenant_id; tenants can only query own logs; platform admins query all | P0 | REQ-FN-805 | TC-MT-070 |
| **REQ-MT-009** | Backup Isolation | Per-tenant backup encryption with tenant DEK; restore scoped to tenant; no cross-tenant restore | P0 | REQ-NFR-003 | TC-MT-080 |
| **REQ-MT-010** | Data Residency Enforcement | Tenant region locked at provisioning; all resources (DB, Redis, S3, compute) deployed in same region; migration requires explicit workflow | P0 | REQ-NFR-025 | TC-MT-090, TC-SEC-150 |

---

## 13. Performance & Scalability Requirements

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-PF-001** | Cold Start Latency | Serverless functions (if any) cold start < 2 s; provisioned concurrency for critical paths | P1 | DOC-06 | TC-PERF-100 |
| **REQ-PF-002** | Database Query Performance | All tenant-scoped queries use covering indexes; p95 < 100 ms; explain plan reviewed in CI | P0 | DOC-06 | TC-PERF-110 |
| **REQ-PF-003** | Cache Strategy | Redis caching for: tenant config (TTL 5 min), user permissions (TTL 1 min), asset lookups (TTL 30 s); cache-aside pattern | P0 | DOC-06 | TC-PERF-120 |
| **REQ-PF-004** | Pagination & Cursors | All list APIs use cursor-based pagination (opaque cursor, max 100 items); no offset pagination | P0 | DOC-07 | TC-PERF-130 |
| **REQ-PF-005** | Bulk Operations | Bulk endpoints support up to 1000 items per request; async processing with job status polling | P0 | DOC-07 | TC-PERF-140 |
| **REQ-PF-006** | WebSocket Scalability | Real-time updates (discovery progress, notifications) via WebSocket; connection multiplexing; ≤ 10k connections/pod | P1 | DOC-09 | TC-PERF-150 |
| **REQ-PF-007** | CDN & Static Assets | Frontend assets served via CDN (Cloudflare/CloudFront); cache-control immutable for hashed assets; Brotli compression | P0 | DOC-09 | TC-PERF-160 |

---

## 14. Observability & Operability Requirements

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-OB-001** | Health Endpoints | Every service exposes `/health/live` (liveness) and `/health/ready` (readiness) for k8s probes | P0 | DOC-06 | TC-OBS-100 |
| **REQ-OB-002** | Structured Metrics | Prometheus metrics: `http_requests_total`, `http_request_duration_seconds`, `db_query_duration_seconds`, `queue_depth`, `cache_hit_ratio`, business metrics | P0 | REQ-NFR-030 | TC-OBS-110 |
| **REQ-OB-003** | Correlation IDs | All requests carry `X-Request-ID` (UUID v7); propagated via headers, logs, traces; generated at edge if missing | P0 | REQ-NFR-032 | TC-OBS-120 |
| **REQ-OB-004** | Error Budget Alerting | SLO-based alerts: burn rate (1h/6h/24h); multi-window, multi-burn-rate; alert on 2% budget consumed in 1h | P0 | DOC-13 | TC-OBS-130 |
| **REQ-OB-005** | Runbook Links | Every critical alert includes runbook URL; runbooks versioned in Git; tested quarterly | P0 | DOC-15 | TC-OBS-140 |
| **REQ-OB-006** | Capacity Planning | Automated capacity forecasts: CPU, memory, disk, connections, custom metrics; 30-day horizon; alert at 80% projected | P1 | DOC-15 | TC-OBS-150 |

---

## 15. Data & Compliance Requirements

| ID | Title | Statement | Priority | Source | Verification |
|----|-------|-----------|----------|--------|--------------|
| **REQ-DC-001** | Data Retention | Configurable per classification: Public (indefinite), Internal (7 yr), Confidential (7 yr + legal hold), Restricted (legal requirement) | P0 | BR-DG-001..010 | TC-DC-001 |
| **REQ-DC-002** | Right to Erasure (GDPR Art. 17) | Tenant admin can trigger erasure: system deletes personal data within 30 days; retains audit logs (legal basis); issues certificate | P0 | UC-TN-006 | TC-DC-010, TC-SEC-030 |
| **REQ-DC-003** | Data Portability (GDPR Art. 20) | Export in machine-readable format (JSON/CSV); includes all user-generated data; delivered via secure link | P0 | UC-TN-006 | TC-DC-020 |
| **REQ-DC-004** | Records of Processing (RoPA) | System maintains RoPA: purposes, categories, recipients, transfers, retention, security measures; exportable for DPA | P0 | DOC-12 | TC-DC-030 |
| **REQ-DC-005** | Data Processing Agreement (DPA) | Standard DPA available for all tenants; custom DPA for Enterprise; signed electronically | P0 | DOC-12 | TC-DC-040 |
| **REQ-DC-006** | Cross-Border Transfer | SCCs (Standard Contractual Clauses) for non-adequate countries; transfer impact assessment documented | P1 | REQ-NFR-025 | TC-DC-050 |
| **REQ-DC-007** | Backup Encryption & Integrity | Backups encrypted with tenant DEK; integrity verified via SHA-256 manifest; restore tested quarterly | P0 | REQ-NFR-003 | TC-DC-060, TC-OBS-050 |
| **REQ-DC-008** | Immutable Audit Trail | Audit logs append-only; Merkle tree root published hourly to transparency log; inclusion proofs verifiable | P0 | REQ-FN-705 | TC-DC-070, TC-SEC-080 |

---

## 16. Traceability Matrices

### 16.1 Capability → Functional Requirements

| Capability | Functional Requirements (REQ-FN-###) |
|------------|--------------------------------------|
| **CAP-010** | REQ-FN-401..414 (Lansweeper), REQ-FN-301..315 (Asset Core) |
| **CAP-011** | REQ-FN-401..403, 404..409 |
| **CAP-016** | REQ-FN-410..412, REQ-FN-315 |
| **CAP-017** | REQ-FN-312..314 |
| **CAP-018** | REQ-FN-305..309 |
| **CAP-020** | REQ-FN-301..311 |
| **CAP-030** | REQ-FN-701..710 |
| **CAP-031** | REQ-FN-701..703 |
| **CAP-033** | REQ-FN-704..706, REQ-FN-805 |
| **CAP-040** | REQ-FN-501..515 |
| **CAP-050** | REQ-FN-201..213, REQ-FN-101..121 |
| **CAP-051** | REQ-FN-201..202 |
| **CAP-053** | REQ-MT-001..010 |
| **CAP-060** | REQ-FN-401..414, REQ-FN-801..802 |
| **CAP-061** | REQ-FN-401..414 |
| **CAP-070** | REQ-FN-303, 304, 403, 706, 515 |
| **CAP-080** | REQ-FN-801..806, REQ-NFR-001..035 |

### 16.2 Functional Requirement → Test Case (Sample)

| Requirement | Test Cases |
|-------------|------------|
| REQ-FN-101 | TC-AUTH-001, TC-AUTH-002 |
| REQ-FN-108 | TC-AUTH-020, TC-AUTH-021 |
| REQ-FN-201 | TC-TENANT-001..005 |
| REQ-FN-205 | TC-TENANT-020..025, TC-SEC-020 |
| REQ-FN-301 | TC-ASSET-001..010 |
| REQ-FN-404 | TC-LS-020..030 |
| REQ-FN-408 | TC-LS-080..090 |
| REQ-FN-410 | TC-LS-110..120 |
| REQ-FN-504 | TC-BILL-040..050 |
| REQ-FN-704 | TC-AUDIT-040..050 |
| REQ-FN-705 | TC-AUDIT-060..070, TC-SEC-080 |
| REQ-MT-001 | TC-MT-001..010 |

> **Complete bidirectional traceability matrix** (all 200+ requirements × 500+ test cases) auto-generated in CI from `docs/14-appendices/traceability-matrix.xlsx` and published as artifact.

---

## 17. Diagrams

| Diagram ID | Title | Type | Location | Description |
|------------|-------|------|----------|-------------|
| **DIA-011** | Requirements Hierarchy | Tree | Section 1 | FN/NFR/SEC/MT/PF/OB/DC taxonomy |
| **DIA-012** | Auth Flow (Username + MFA) | Sequence | Section 2.1 | REQ-FN-101..104 |
| **DIA-013** | SSO Flow (SAML/OIDC) | Sequence | Section 2.1 | REQ-FN-105..107 |
| **DIA-014** | JWT Token Lifecycle | State | Section 2.1 | REQ-FN-108..110 |
| **DIA-015** | RBAC/ABAC Decision Flow | Flowchart | Section 2.2 | REQ-FN-111..115 |
| **DIA-016** | Tenant Provisioning Sequence | Sequence | Section 3.1 | REQ-FN-201..202 |
| **DIA-017** | Tenant Offboarding Flow | Flowchart | Section 3.1 | REQ-FN-205..206 |
| **DIA-018** | Asset Lifecycle State Machine | State | Section 4.1 | REQ-FN-305..309 |
| **DIA-019** | Lansweeper Sync Architecture | Component | Section 5.2 | REQ-FN-404..409 |
| **DIA-020** | Field Ownership Matrix | Matrix | Section 5.3 | REQ-FN-410..412 |
| **DIA-021** | Invoice Generation Flow | Sequence | Section 6.2 | REQ-FN-504..507 |
| **DIA-022** | Cost Allocation Model | Diagram | Section 6.3 | REQ-FN-508..510 |
| **DIA-023** | Notification Delivery Guarantees | Sequence | Section 7 | REQ-FN-606 |
| **DIA-024** | Compliance Assessment Flow | Flowchart | Section 8.2 | REQ-FN-704..706 |
| **DIA-025** | Multi-Tenancy Isolation Layers | Layered | Section 12 | REQ-MT-001..010 |
| **DIA-026** | Observability Stack | Component | Section 14 | REQ-OB-001..006 |

---

## 18. Tables

| Table ID | Title | Location | Row Count |
|----------|-------|----------|-----------|
| **TBL-028** | Requirements Identifier Scheme | Section 1.1 | 7 |
| **TBL-029** | EARS Notation Patterns | Section 1.2 | 5 |
| **TBL-030** | Authentication Requirements | Section 2.1 | 10 |
| **TBL-031** | Authorization Requirements | Section 2.2 | 5 |
| **TBL-032** | User/Session Management Requirements | Section 2.3 | 6 |
| **TBL-033** | Tenant Lifecycle Requirements | Section 3.1 | 6 |
| **TBL-034** | Tenant Configuration Requirements | Section 3.2 | 7 |
| **TBL-035** | Asset Registration Requirements | Section 4.1 | 8 |
| **TBL-036** | Maintenance Requirements | Section 4.2 | 3 |
| **TBL-037** | CMDB Requirements | Section 4.3 | 4 |
| **TBL-038** | Lansweeper Connection Requirements | Section 5.1 | 3 |
| **TBL-039** | Sync Requirements | Section 5.2 | 6 |
| **TBL-040** | Field Ownership Requirements | Section 5.3 | 3 |
| **TBL-041** | Software/Vuln Requirements | Section 5.4 | 2 |
| **TBL-042** | Subscription Requirements | Section 6.1 | 3 |
| **TBL-043** | Invoicing Requirements | Section 6.2 | 4 |
| **TBL-044** | Cost Allocation Requirements | Section 6.3 | 3 |
| **TBL-045** | Contracts/True-Up Requirements | Section 6.4 | 5 |
| **TBL-046** | Notification Requirements | Section 7 | 6 |
| **TBL-047** | Framework/Policy Requirements | Section 8.1 | 3 |
| **TBL-048** | Assessment/Evidence Requirements | Section 8.2 | 3 |
| **TBL-049** | Risk/Incident Requirements | Section 8.3 | 4 |
| **TBL-050** | Core Platform Requirements | Section 9 | 6 |
| **TBL-051** | Availability & Reliability NFRs | Section 10.1 | 6 |
| **TBL-052** | Performance & Scalability NFRs | Section 10.2 | 8 |
| **TBL-053** | Security & Privacy NFRs | Section 10.3 | 6 |
| **TBL-054** | Observability NFRs | Section 10.4 | 6 |
| **TBL-055** | Security Requirements | Section 11 | 10 |
| **TBL-056** | Multi-Tenancy Requirements | Section 12 | 10 |
| **TBL-057** | Performance Requirements | Section 13 | 7 |
| **TBL-058** | Observability Requirements | Section 14 | 6 |
| **TBL-059** | Data & Compliance Requirements | Section 15 | 8 |
| **TBL-060** | Capability → FN Requirements | Section 16.1 | 14 |
| **TBL-061** | Requirement → Test Case (Sample) | Section 16.2 | 14 |
| **TBL-062** | Diagram Index | Section 17 | 16 |
| **TBL-063** | Table Index | Section 18 | 36 |

---

---

**End of DOC-03 — NexusAsset Cloud Requirements Specification**

*Document Control: Version 1.0.0 | Classification: Internal — Confidential | Next Review: 2026-12-09*

*For questions or updates: docs@nexusasset.cloud | Architecture Review Board: arch-review@nexusasset.cloud*