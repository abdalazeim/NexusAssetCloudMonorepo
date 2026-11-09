╔════════════════════════════════════════════════════════════════════════════════════════════════╗
║ Document ID: DOC-02                                                                            ║
║ Title: Business Analysis — Enterprise Capabilities, Processes, Use Cases & Governance          ║
║ Version: 1.0.0                                                                                 ║
║ Status: ✓ Approved                                                                             ║
║ Classification: Internal — Confidential                                                        ║
║ Authors: NexusAsset Cloud Architecture Team                                                    ║
║ Reviewers: Product Management, Security, Compliance, Legal                                     ║
║ Approved By: CTO / CISO / VP Product                                                           ║
║ Date: 2026-11-09                                                                               ║
║ Supersedes: —                                                                                  ║
║ Related: DOC-00, DOC-01, DOC-03, DOC-04, DOC-05, DOC-06, DOC-07, DOC-08, DOC-12, DOC-15      ║
║ Traceability: REQ-FN-### ← CAP-###, REQ-NFR-### ← CAP-###, TC-### ← UC-###                    ║
╚════════════════════════════════════════════════════════════════════════════════════════════════╝

# ═══════════════════════════════════════════════════════════════
# NexusAsset Cloud — Business Analysis (DOC-02)
# ═══════════════════════════════════════════════════════════════

> **Executive Summary** — This document defines the enterprise business capabilities, core processes, use cases, governance frameworks, compliance requirements, audit mandates, key performance indicators, and stakeholder value mappings for NexusAsset Cloud. It serves as the authoritative business specification from which all functional (REQ-FN-###) and non-functional (REQ-NFR-###) requirements in DOC-03 are derived. Every capability (CAP-###), process (PROC-###), use case (UC-###), and rule (BR-###) carries a stable identifier for end-to-end traceability to architecture (DOC-04–DOC-06), API design (DOC-07), test cases (DOC-13), and operational runbooks (DOC-15).

---

## Table of Contents

1. [Business Capability Model](#1-business-capability-model)
2. [Core Business Processes](#2-core-business-processes)
3. [Use Case Catalog per Bounded Context](#3-use-case-catalog-per-bounded-context)
4. [Governance & Compliance Framework](#4-governance--compliance-framework)
5. [Audit Requirements & Evidence Generation](#5-audit-requirements--evidence-generation)
6. [Key Performance Indicators (KPIs) & Success Metrics](#6-key-performance-indicators-kpis--success-metrics)
7. [Stakeholder Value Mapping](#7-stakeholder-value-mapping)
8. [Business Rules Catalog](#8-business-rules-catalog)
9. [Risk & Assumption Register (Business View)](#9-risk--assumption-register-business-view)
10. [Traceability Matrix](#10-traceability-matrix)
11. [Diagrams](#11-diagrams)
12. [Tables](#12-tables)

---

## 1. Business Capability Model

### 1.1 Capability Taxonomy

NexusAsset Cloud capabilities are organized into a three-level hierarchy aligned with the **Business Capability Modeling** framework (TOGAF® / BIZBOK®). Each capability maps to one or more bounded contexts (see DOC-04) and traces to functional requirements (REQ-FN-###).

```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                         NEXUSASSET CLOUD CAPABILITY MAP (LEVEL 1)                          │
├─────────────────────┬─────────────────────┬─────────────────────┬───────────────────────────┤
│   CAP-010           │   CAP-020           │   CAP-030           │   CAP-040                 │
│   Asset Discovery   │   Asset Lifecycle   │   Governance &      │   Financial &             │
│   & Inventory       │   Management        │   Compliance        │   Contract Management     │
├─────────────────────┼─────────────────────┼─────────────────────┼───────────────────────────┤
│   CAP-050           │   CAP-060           │   CAP-070           │   CAP-080                 │
│   Tenant & Identity │   Integration &     │   Reporting &       │   Platform Operations     │
│   Management        │   Ecosystem         │   Analytics         │   & Observability         │
└─────────────────────┴─────────────────────┴─────────────────────┴───────────────────────────┘
```

> **Note** — Capabilities CAP-010 through CAP-080 are **Level 1** (Strategic). Level 2 (Tactical) and Level 3 (Operational) decompositions follow in Sections 1.2–1.9. Each Level 2/3 capability carries a stable ID (e.g., `CAP-011`, `CAP-011.1`).

---

### 1.2 CAP-010: Asset Discovery & Inventory

| Capability ID | Name | Description | Bounded Context | Priority |
|--------------|------|-------------|-----------------|----------|
| **CAP-011** | Network-Based Discovery | Automated discovery of IT assets via Lansweeper scanning (agentless/agent-based), SNMP, WMI, SSH, ARP, CDP/LLDP | `nxa_lansweeper` | P0 |
| **CAP-012** | Cloud Asset Discovery | Discovery of cloud resources (AWS/Azure/GCP) via provider APIs, CloudFormation/Terraform state, CSPM integrations | `nxa_assets` | P1 |
| **CAP-013** | Virtualization Discovery | Discovery of VMs, containers, clusters (VMware, Hyper-V, Kubernetes, OpenShift) via vCenter, K8s API | `nxa_assets` | P1 |
| **CAP-014** | Software Asset Discovery | Installed software enumeration, license recognition, version tracking, vulnerability correlation (CVE) | `nxa_assets` | P0 |
| **CAP-015** | Hardware Asset Discovery | Physical hardware inventory (servers, network devices, peripherals) via IPMI, Redfish, SNMP | `nxa_assets` | P1 |
| **CAP-016** | Asset Reconciliation & Deduplication | Multi-source correlation, fuzzy matching, confidence scoring, golden record creation | `nxa_assets` | P0 |
| **CAP-017** | CMDB Population & CI Relationship Mapping | Configuration Item (CI) creation, relationship discovery (depends-on, hosts, connects-to), service mapping | `nxa_assets` | P0 |
| **CAP-018** | Asset Lifecycle State Management | State machine: Discovered → Authorized → Active → Maintenance → Retired → Disposed | `nxa_assets` | P0 |

> **Traceability**: CAP-011 → REQ-FN-011, REQ-FN-012, REQ-FN-013; CAP-016 → REQ-FN-016, REQ-FN-017; CAP-017 → REQ-FN-018, REQ-FN-019

---

### 1.3 CAP-020: Asset Lifecycle Management

| Capability ID | Name | Description | Bounded Context | Priority |
|--------------|------|-------------|-----------------|----------|
| **CAP-021** | Asset Registration & Onboarding | Manual/automated asset registration, barcode/QR tagging, ownership assignment, location tracking | `nxa_assets` | P0 |
| **CAP-022** | Asset Assignment & Custody Transfer | User/device assignment, checkout/check-in, custody chain, responsible party tracking | `nxa_assets` | P0 |
| **CAP-023** | Maintenance & Service Management | Preventive/corrective maintenance scheduling, work orders, SLA tracking, vendor coordination | `nxa_assets` | P1 |
| **CAP-024** | Asset Relocation & Movement Tracking | Physical/logical moves, data center migrations, geo-tracking, audit trail | `nxa_assets` | P1 |
| **CAP-025** | Retirement & Disposal Management | Data sanitization (NIST 800-88), disposal certificates, environmental compliance (WEEE), asset write-off | `nxa_assets` | P1 |
| **CAP-026** | Warranty & Support Contract Tracking | Vendor warranty, support entitlements, renewal alerts, cost optimization | `nxa_assets` | P2 |
| **CAP-027** | Asset Financial Tracking | Capitalization, depreciation (straight-line, declining balance), TCO calculation, budget allocation | `nxa_billing` | P1 |

---

### 1.4 CAP-030: Governance & Compliance

| Capability ID | Name | Description | Bounded Context | Priority |
|--------------|------|-------------|-----------------|----------|
| **CAP-031** | Policy Definition & Enforcement | Policy-as-code (OPA/Rego), automated compliance checks, drift detection, remediation workflows | `nxa_audit`, `nxa_core` | P0 |
| **CAP-032** | Regulatory Compliance Mapping | Mapping to ISO 27001, SOC 2, GDPR, HIPAA, PCI-DSS, NIST CSF, CIS Controls | `nxa_audit` | P0 |
| **CAP-033** | Audit Trail & Evidence Collection | Immutable audit logs, tamper-evident storage, evidence packages, chain of custody | `nxa_audit` | P0 |
| **CAP-034** | Risk Assessment & Treatment | Risk register, likelihood/impact scoring, treatment plans, residual risk tracking | `nxa_audit` | P1 |
| **CAP-035** | Data Governance & Classification | Data catalog, sensitivity labels (Public/Internal/Confidential/Restricted), retention policies | `nxa_core`, `nxa_audit` | P1 |
| **CAP-036** | Vendor & Third-Party Risk Management | Vendor assessments, questionnaires, continuous monitoring, contract compliance | `nxa_audit` | P2 |
| **CAP-037** | Incident & Breach Management | Incident response playbooks, breach notification timelines, forensic evidence preservation | `nxa_audit`, `nxa_notifications` | P0 |

---

### 1.5 CAP-040: Financial & Contract Management

| Capability ID | Name | Description | Bounded Context | Priority |
|--------------|------|-------------|-----------------|----------|
| **CAP-041** | Subscription & License Management | SaaS subscriptions, perpetual licenses, concurrent/float licensing, true-up automation | `nxa_billing` | P0 |
| **CAP-042** | Cost Allocation & Chargeback | Multi-tenant cost allocation (proportional, fixed, tiered), showback/chargeback reports, budget vs actual | `nxa_billing` | P0 |
| **CAP-043** | Invoice Generation & Billing Automation | Recurring/in usage-based invoicing, multi-currency, tax calculation, payment gateway integration | `nxa_billing` | P0 |
| **CAP-044** | Contract Lifecycle Management | Contract authoring, negotiation, approval workflows, renewal management, obligation tracking | `nxa_billing` | P1 |
| **CAP-045** | Financial Reporting & Analytics | P&L by tenant/service, margin analysis, forecasting, anomaly detection | `nxa_billing` | P1 |

---

### 1.6 CAP-050: Tenant & Identity Management

| Capability ID | Name | Description | Bounded Context | Priority |
|--------------|------|-------------|-----------------|----------|
| **CAP-051** | Multi-Tenant Onboarding & Provisioning | Self-service tenant creation, schema provisioning, default policies, branding, domain configuration | `nxa_tenants`, `nxa_accounts` | P0 |
| **CAP-052** | Identity & Access Management (IAM) | RBAC/ABAC, SSO (SAML/OIDC), MFA, SCIM provisioning, just-in-time access | `nxa_accounts` | P0 |
| **CAP-053** | Tenant Isolation & Data Segregation | Row-Level Security (RLS), schema-per-tenant, encryption-per-tenant, network segmentation | `nxa_tenants`, `nxa_core` | P0 |
| **CAP-054** | Tenant Lifecycle & Offboarding | Suspension, data export, secure deletion, regulatory hold, grace periods | `nxa_tenants` | P1 |
| **CAP-055** | Delegated Administration | Tenant admin roles, custom role builder, permission boundaries, audit of admin actions | `nxa_accounts` | P1 |

---

### 1.7 CAP-060: Integration & Ecosystem

| Capability ID | Name | Description | Bounded Context | Priority |
|--------------|------|-------------|-----------------|----------|
| **CAP-061** | Lansweeper Deep Integration | GraphQL API sync, webhook consumption, delta detection, field ownership matrix, conflict resolution | `nxa_lansweeper` | P0 |
| **CAP-062** | ITSM Integration (ServiceNow, Jira, etc.) | Incident/change/problem synchronization, CMDB push/pull, bi-directional workflow | `nxa_assets`, `nxa_audit` | P1 |
| **CAP-063** | SIEM/SOAR Integration | Log forwarding (CEF/LEEF/JSON), alert enrichment, automated response playbooks | `nxa_audit`, `nxa_notifications` | P1 |
| **CAP-064** | API Gateway & Developer Portal | OpenAPI 3.1 publication, rate limiting, API keys, usage analytics, sandbox environment | `nxa_core` | P0 |
| **CAP-065** | Webhook & Event Streaming | Outbound webhooks (retry, dead-letter), Kafka/EventBridge integration, event schema registry | `nxa_core`, `nxa_notifications` | P1 |

---

### 1.8 CAP-070: Reporting & Analytics

| Capability ID | Name | Description | Bounded Context | Priority |
|--------------|------|-------------|-----------------|----------|
| **CAP-071** | Operational Dashboards | Real-time asset health, discovery status, sync metrics, compliance posture | `nxa_assets`, `nxa_lansweeper` | P0 |
| **CAP-072** | Executive & Compliance Reports | Board-ready reports, audit evidence packages, regulatory submissions, trend analysis | `nxa_audit`, `nxa_billing` | P0 |
| **CAP-073** | Ad-Hoc Query & Self-Service BI | Natural language query, saved views, scheduled exports, embedding (iframe/React) | `nxa_assets` | P1 |
| **CAP-074** | Predictive Analytics & ML | Asset failure prediction, license optimization, anomaly detection, capacity forecasting | `nxa_assets`, `nxa_billing` | P2 |

---

### 1.9 CAP-080: Platform Operations & Observability

| Capability ID | Name | Description | Bounded Context | Priority |
|--------------|------|-------------|-----------------|----------|
| **CAP-081** | Infrastructure Provisioning & GitOps | Terraform/Ansible, ArgoCD/Flux, environment promotion, drift reconciliation | Platform (Infra) | P0 |
| **CAP-082** | Release Management & Deployment | Blue/green, canary, feature flags, automated rollback, deployment windows | Platform (CI/CD) | P0 |
| **CAP-083** | Observability & Alerting | Metrics (Prometheus), logs (Loki), traces (Tempo), dashboards (Grafana), SLO-based alerting | Platform (Obs) | P0 |
| **CAP-084** | Backup, DR & Business Continuity | RPO/RTO targets, point-in-time recovery, cross-region replication, chaos engineering | Platform (Data) | P0 |
| **CAP-085** | Security Operations | Vulnerability scanning, secret rotation, certificate management, threat detection | Platform (Sec) | P0 |

---

## 2. Core Business Processes

### 2.1 Process Architecture Overview

Business processes are modeled using **BPMN 2.0** notation and mapped to bounded contexts. Each process (PROC-###) decomposes into activities, decision gateways, and events. Processes are triggered by **commands** (CMD-###) and emit **domain events** (EVT-###).

```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                           CORE BUSINESS PROCESS MAP                                          │
├─────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                             │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐        │
│   │  PROC-010    │────▶│  PROC-020    │────▶│  PROC-030    │────▶│  PROC-040    │        │
│   │  Discovery   │     │  Reconcile   │     │  Authorize   │     │  Lifecycle   │        │
│   │  & Ingest    │     │  & Enrich    │     │  & Onboard   │     │  Management  │        │
│   └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘        │
│         │                    │                    │                    │                   │
│         ▼                    ▼                    ▼                    ▼                   │
│   ┌──────────────────────────────────────────────────────────────────────────────────┐   │
│   │                    PROC-050: Governance, Compliance & Audit                      │   │
│   │         (Continuous — runs in parallel, evaluates all asset states)              │   │
│   └──────────────────────────────────────────────────────────────────────────────────┘   │
│         │                    │                    │                    │                   │
│         ▼                    ▼                    ▼                    ▼                   │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐        │
│   │  PROC-060    │     │  PROC-070    │     │  PROC-080    │     │  PROC-090    │        │
│   │  Financial   │     │  Reporting   │     │  Tenant      │     │  Platform    │        │
│   │  Management  │     │  & Analytics │     │  Operations  │     │  Operations  │        │
│   └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘        │
│                                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

### 2.2 PROC-010: Discovery & Ingestion

**Trigger**: Scheduled (cron), webhook (Lansweeper), API call (cloud provider), manual initiation

**Activities**:
1. **A-010.1** — Validate scan configuration (credentials, targets, schedules) — *CAP-011, CAP-012*
2. **A-010.2** — Execute discovery scan (Lansweeper GraphQL, cloud APIs, SNMP/SSH) — *CAP-011–015*
3. **A-010.3** — Normalize raw payload to canonical Asset DTO (see DOC-04 VO-010) — *CAP-016*
4. **A-010.4** — Emit `AssetDiscovered` domain event (EVT-010) with correlation ID — *CAP-011–015*
5. **A-010.5** — Persist raw scan artifact to object storage (audit/forensics) — *CAP-033*
6. **A-010.6** — Update discovery metrics (assets found, duration, error rate) — *CAP-071*

**Decision Gateways**:
- **GW-010.1**: Scan success? → No: emit `DiscoveryFailed` (EVT-011), alert on-call, retry with backoff
- **GW-010.2**: Tenant quota exceeded? → Yes: throttle, notify tenant admin, emit `QuotaExceeded` (EVT-012)

**Output Events**: `AssetDiscovered` (EVT-010), `DiscoveryCompleted` (EVT-013), `DiscoveryFailed` (EVT-011)

**SLA**: P0 scans ≤ 30 min; P1 scans ≤ 4 hr; P2 scans ≤ 24 hr

---

### 2.3 PROC-020: Reconciliation & Enrichment

**Trigger**: `AssetDiscovered` (EVT-010) event stream (Kafka topic `nxa.assets.discovered`)

**Activities**:
1. **A-020.1** — Fetch existing asset candidates by matching keys (MAC, serial, BIOS UUID, cloud resource ID) — *CAP-016*
2. **A-020.2** — Execute fuzzy matching algorithm (Jaro-Winkler, token set ratio) with confidence threshold ≥ 0.85 — *CAP-016*
3. **A-020.3** — Apply **Field Ownership Matrix** (see DOC-08 Table 3) to resolve conflicts: Lansweeper wins for HW, Cloud API wins for cloud tags, Manual wins for owner/location — *CAP-016*
4. **A-020.4** — Enrich with external data: CVE (NVD), EOL dates, vendor support, pricing — *CAP-014, CAP-026*
5. **A-020.5** — Build/update CI relationship graph (depends-on, hosts, connects-to) via network topology analysis — *CAP-017*
6. **A-020.6** — Emit `AssetReconciled` (EVT-020) with golden record ID and merge decisions — *CAP-016, CAP-017*

**Decision Gateways**:
- **GW-020.1**: High-confidence match (≥ 0.95)? → Auto-merge
- **GW-020.2**: Medium-confidence (0.85–0.95)? → Queue for steward review (human-in-the-loop)
- **GW-020.3**: No match? → Create new CI candidate

**Output Events**: `AssetReconciled` (EVT-020), `ReconciliationConflict` (EVT-021), `CIrelationshipUpdated` (EVT-022)

---

### 2.4 PROC-030: Authorization & Onboarding

**Trigger**: `AssetReconciled` (EVT-020) + tenant policy evaluation

**Activities**:
1. **A-030.1** — Evaluate authorization policy (OPA): Is asset type allowed? Is tenant quota available? — *CAP-031, CAP-053*
2. **A-030.2** — Assign default lifecycle state: `Authorized` — *CAP-018*
3. **A-030.3** — Apply tenant-specific tagging/labeling policies — *CAP-035*
4. **A-030.4** — Create CMDB record with relationships — *CAP-017*
5. **A-030.5** — Trigger notification to asset owner/stakeholders — *CAP-052, CAP-065*
6. **A-030.6** — Emit `AssetAuthorized` (EVT-030) — *CAP-018*

**Decision Gateways**:
- **GW-030.1**: Policy deny? → Emit `AuthorizationDenied` (EVT-031), route to exception workflow
- **GW-030.2**: Requires approval? → Create approval task (human-in-the-loop), emit `ApprovalRequired` (EVT-032)

**Output Events**: `AssetAuthorized` (EVT-030), `AuthorizationDenied` (EVT-031), `ApprovalRequired` (EVT-032)

---

### 2.5 PROC-040: Lifecycle Management

**Trigger**: State transition commands (CMD-040.x), scheduled maintenance, user actions

**State Machine** (per CAP-018):

```
┌─────────────┐     CMD-040.1      ┌─────────────┐     CMD-040.2      ┌─────────────┐
│  Discovered │ ─────────────────▶ │ Authorized  │ ─────────────────▶ │   Active    │
└─────────────┘   (auto/policy)    └─────────────┘   (assignment)     └──────┬──────┘
      ▲                                                                      │
      │                                                                      │
      │ CMD-040.6 (restore)                                                  │ CMD-040.3 (maintenance)
      │                                                                      ▼
┌─────────────┐     CMD-040.5      ┌─────────────┐     CMD-040.4      ┌─────────────┐
│  Disposed   │ ◀───────────────── │  Retired    │ ◀───────────────── │ Maintenance │
└─────────────┘   (final)          └─────────────┘   (end-of-life)    └─────────────┘
```

**Activities per Transition**:
- **CMD-040.1** (Discovered → Authorized): Policy eval, quota check, tagging — *PROC-030*
- **CMD-040.2** (Authorized → Active): Assignment to user/location, warranty activation — *CAP-022*
- **CMD-040.3** (Active → Maintenance): Work order creation, SLA timer start, replacement provision — *CAP-023*
- **CMD-040.4** (Maintenance → Retired): Data sanitization verification, disposal certificate — *CAP-025*
- **CMD-040.5** (Retired → Disposed): Physical disposal, financial write-off, CMDB cleanup — *CAP-025, CAP-027*
- **CMD-040.6** (Retired → Active): Re-commissioning, re-certification, new warranty — *CAP-023*

**Events Emitted**: `AssetStateChanged` (EVT-040) with from/to state, reason, actor

---

### 2.6 PROC-050: Governance, Compliance & Audit (Continuous)

**Trigger**: Continuous evaluation via policy engine (OPA), scheduled scans, event-driven (EVT-###)

**Activities**:
1. **A-050.1** — Evaluate all assets against policy bundle (CIS, custom) every 15 min — *CAP-031*
2. **A-050.2** — Detect drift: desired state (policy) vs actual state (CMDB) — *CAP-031*
3. **A-050.3** — Generate compliance findings (PASS/FAIL/WARN) with evidence — *CAP-032, CAP-033*
4. **A-050.4** — Auto-remediate where policy allows (e.g., tag correction, encryption enable) — *CAP-031*
5. **A-050.5** — Create remediation tickets for non-auto-remediable findings — *CAP-034, CAP-062*
6. **A-050.6** — Produce audit evidence packages (immutable, signed) on schedule or demand — *CAP-033*
7. **A-050.7** — Update compliance posture dashboards and executive reports — *CAP-072*

**Decision Gateways**:
- **GW-050.1**: Critical finding (CVSS ≥ 9.0 or data exposure)? → Immediate alert, incident creation — *CAP-037*
- **GW-050.2**: Remediation overdue (> SLA)? → Escalation to tenant admin → CISO — *CAP-034*

---

### 2.7 PROC-060: Financial Management

**Trigger**: Asset lifecycle events (EVT-040), billing cycles (monthly), contract events

**Activities**:
1. **A-060.1** — Calculate asset-level cost (depreciation, support, licensing, cloud spend) — *CAP-027, CAP-041*
2. **A-060.2** — Allocate costs to tenants/cost centers per allocation model — *CAP-042*
3. **A-060.3** — Generate invoices (recurring + usage) with tax — *CAP-043*
4. **A-060.4** — Process payments, handle dunning, reconcile — *CAP-043*
5. **A-060.5** — Contract renewal forecasting, true-up calculations — *CAP-044*
6. **A-060.6** — Financial reporting (P&L, margin, forecast) — *CAP-045*

---

### 2.8 PROC-070: Reporting & Analytics

**Trigger**: Scheduled (daily/weekly/monthly), ad-hoc API, dashboard subscriptions

**Activities**:
1. **A-070.1** — Aggregate operational metrics (discovery success rate, sync latency, asset counts) — *CAP-071*
2. **A-070.2** — Generate compliance posture reports per framework (ISO 27001, SOC 2) — *CAP-072*
3. **A-070.3** — Execute ad-hoc queries (SQL/GraphQL) with tenant isolation — *CAP-073*
4. **A-070.4** — Run ML models for predictions (failure, optimization) — *CAP-074*
5. **A-070.5** — Distribute reports (email, portal, webhook, S3) — *CAP-065*

---

### 2.9 PROC-080: Tenant Operations

**Trigger**: Tenant admin actions, platform operations, lifecycle events

**Activities**:
1. **A-080.1** — Tenant provisioning: schema, RLS policies, default roles, branding — *CAP-051*
2. **A-080.2** — Tenant configuration: custom fields, workflows, integrations, policies — *CAP-051, CAP-031*
3. **A-080.3** — Tenant monitoring: usage, quota, health, compliance — *CAP-053, CAP-071*
4. **A-080.4** — Tenant support: impersonation (audited), data export, migration — *CAP-054, CAP-055*
5. **A-080.5** — Tenant offboarding: hold, export, secure delete, certificate — *CAP-054*

---

### 2.10 PROC-090: Platform Operations

**Trigger**: CI/CD pipeline, infrastructure changes, incidents, capacity thresholds

**Activities**:
1. **A-090.1** — Infrastructure deployment via GitOps (ArgoCD) — *CAP-081*
2. **A-090.2** — Application deployment (blue/green, canary) with automated rollback — *CAP-082*
3. **A-090.3** — Observability stack management (dashboards, alerts, SLOs) — *CAP-083*
4. **A-090.4** — Backup verification, DR drills, chaos experiments — *CAP-084*
5. **A-090.5** — Security patching, secret rotation, cert renewal — *CAP-085*

---

## 3. Use Case Catalog per Bounded Context

### 3.1 Use Case Identification Scheme

| Prefix | Bounded Context | Domain |
|--------|----------------|--------|
| **UC-AC** | `nxa_accounts` | Identity & Access |
| **UC-TN** | `nxa_tenants` | Tenant Management |
| **UC-AS** | `nxa_assets` | Asset Core |
| **UC-LS** | `nxa_lansweeper` | Lansweeper Integration |
| **UC-BL** | `nxa_billing` | Billing & Contracts |
| **UC-NT** | `nxa_notifications` | Notifications |
| **UC-AU** | `nxa_audit` | Audit & Compliance |
| **UC-CR** | `nxa_core` | Core Platform |

Each use case follows the template: **UC-XX-###: [Verb] [Noun] — [Brief Outcome]**

---

### 3.2 nxa_accounts (Identity & Access)

| Use Case ID | Name | Actor | Description | Priority | Trace |
|------------|------|-------|-------------|----------|-------|
| **UC-AC-001** | Authenticate User | End User | Username/password + MFA (TOTP/WebAuthn), JWT issuance (access + refresh) | P0 | REQ-FN-101 |
| **UC-AC-002** | SSO Login (SAML/OIDC) | End User | Federated login via IdP (Azure AD, Okta, Google), JIT provisioning | P0 | REQ-FN-102 |
| **UC-AC-003** | Manage User Profile | End User | Update profile, avatar, preferences, notification settings | P1 | REQ-FN-103 |
| **UC-AC-004** | Manage API Keys | Developer | Create/revoke/scopes API keys, rotation, usage analytics | P0 | REQ-FN-104 |
| **UC-AC-005** | Assign Roles & Permissions | Tenant Admin | RBAC assignment, custom roles, permission boundaries, delegation | P0 | REQ-FN-105 |
| **UC-AC-006** | Configure ABAC Policies | Security Admin | Attribute-based policies (resource tags, time, location, risk score) | P1 | REQ-FN-106 |
| **UC-AC-007** | Impersonate User (Audited) | Support Admin | Time-boxed impersonation with full audit trail, requires approval | P1 | REQ-FN-107 |
| **UC-AC-008** | Manage MFA Methods | End User | Register TOTP, WebAuthn, backup codes, recovery | P0 | REQ-FN-108 |
| **UC-AC-009** | Session Management | End User | View/revoke active sessions, device management, concurrent session limits | P1 | REQ-FN-109 |
| **UC-AC-010** | SCIM Provisioning | IdP Admin | SCIM 2.0 endpoint for user/group sync from IdP | P1 | REQ-FN-110 |

---

### 3.3 nxa_tenants (Tenant Management)

| Use Case ID | Name | Actor | Description | Priority | Trace |
|------------|------|-------|-------------|----------|-------|
| **UC-TN-001** | Create Tenant (Self-Service) | Prospect | Sign-up wizard: name, domain, admin user, plan, branding | P0 | REQ-FN-201 |
| **UC-TN-002** | Configure Tenant Settings | Tenant Admin | Custom fields, workflows, retention, integrations, policies | P0 | REQ-FN-202 |
| **UC-TN-003** | Manage Tenant Users & Groups | Tenant Admin | Invite users, assign roles, group management, bulk import (CSV/SCIM) | P0 | REQ-FN-203 |
| **UC-TN-004** | View Tenant Usage & Quotas | Tenant Admin | Real-time dashboards: asset count, API calls, storage, scan frequency | P0 | REQ-FN-204 |
| **UC-TN-005** | Request Quota Increase | Tenant Admin | Workflow: request → approval → provisioning → notification | P1 | REQ-FN-205 |
| **UC-TN-006** | Export Tenant Data | Tenant Admin | GDPR/DSAR export: all tenant data in standard format (JSON/CSV), signed | P0 | REQ-FN-206 |
| **UC-TN-007** | Suspend/Reactivate Tenant | Platform Admin | Graceful suspension (read-only), reactivation, data preservation | P1 | REQ-FN-207 |
| **UC-TN-008** | Offboard Tenant (Secure Delete) | Platform Admin | Regulatory hold check → export → cryptographic erase → certificate | P0 | REQ-FN-208 |
| **UC-TN-009** | Custom Domain & Branding | Tenant Admin | CNAME verification, SSL (Let's Encrypt), logo, colors, favicon | P1 | REQ-FN-209 |
| **UC-TN-010** | Tenant Hierarchy (Parent/Child) | Platform Admin | Multi-level tenancy for MSPs, consolidated billing, policy inheritance | P2 | REQ-FN-210 |

---

### 3.4 nxa_assets (Asset Core)

| Use Case ID | Name | Actor | Description | Priority | Trace |
|------------|------|-------|-------------|----------|-------|
| **UC-AS-001** | Register Asset Manually | Asset Manager | Form-based entry: type, serial, model, owner, location, tags, attachments | P0 | REQ-FN-301 |
| **UC-AS-002** | Bulk Import Assets | Asset Manager | CSV/Excel import with validation, preview, error rollback, template download | P0 | REQ-FN-302 |
| **UC-AS-003** | View Asset Details | Any Authorized | 360° view: properties, relationships, history, compliance, financials, tickets | P0 | REQ-FN-303 |
| **UC-AS-004** | Search & Filter Assets | Any Authorized | Full-text, faceted, saved queries, export, API access | P0 | REQ-FN-304 |
| **UC-AS-005** | Assign/Reassign Asset | Asset Manager | User/location assignment, custody transfer, signature capture, history | P0 | REQ-FN-305 |
| **UC-AS-006** | Schedule Maintenance | Asset Manager | Preventive/corrective, recurring, SLA, technician assignment, parts | P1 | REQ-FN-306 |
| **UC-AS-007** | Record Maintenance Outcome | Technician | Work completion, parts used, downtime, next service date, photos | P1 | REQ-FN-307 |
| **UC-AS-008** | Relocate Asset | Asset Manager | Physical/logical move, data center rack/U position, geo-coordinates | P1 | REQ-FN-308 |
| **UC-AS-009** | Retire & Dispose Asset | Asset Manager | Workflow: sanitize → certify → dispose → write-off → CMDB cleanup | P0 | REQ-FN-309 |
| **UC-AS-010** | Manage CI Relationships | CMDB Admin | Visual topology editor, depends-on/hosts/connects-to, impact analysis | P0 | REQ-FN-310 |
| **UC-AS-011** | Asset Comparison & Diff | Asset Manager | Side-by-side comparison of asset versions, scan vs manual, source diff | P1 | REQ-FN-311 |
| **UC-AS-012** | Custom Fields & Schemas | Tenant Admin | Dynamic custom fields (text, number, date, enum, reference), validation | P1 | REQ-FN-312 |

---

### 3.5 nxa_lansweeper (Lansweeper Integration)

| Use Case ID | Name | Actor | Description | Priority | Trace |
|------------|------|-------|-------------|----------|-------|
| **UC-LS-001** | Configure Lansweeper Connection | Platform Admin | Site/scanning server setup: URL, credentials (PAT/OAuth), sync schedule, site mapping | P0 | REQ-FN-401 |
| **UC-LS-002** | Trigger Manual Sync | Asset Manager | On-demand full/incremental sync for specific site or all sites | P0 | REQ-FN-402 |
| **UC-LS-003** | View Sync Status & History | Asset Manager | Real-time progress, last sync time, assets processed, errors, throughput | P0 | REQ-FN-403 |
| **UC-LS-004** | Configure Field Ownership Matrix | Data Steward | Per-field source priority (Lansweeper/Cloud/Manual), conflict rules, lock fields | P0 | REQ-FN-404 |
| **UC-LS-005** | Resolve Sync Conflicts | Data Steward | Conflict queue: side-by-side view, accept source, merge, mark reviewed | P0 | REQ-FN-405 |
| **UC-LS-006** | Manage Webhook Subscriptions | Platform Admin | Register Lansweeper webhooks (asset created/updated/deleted), retry/DLQ config | P1 | REQ-FN-406 |
| **UC-LS-007** | Delta Sync & Change Detection | System | Automated delta detection via Lansweeper `lastChanged` timestamps, bulk upsert | P0 | REQ-FN-407 |
| **UC-LS-008** | Software License Reconciliation | SAM Manager | Match discovered software to license entitlements, compliance calculation | P1 | REQ-FN-408 |
| **UC-LS-009** | Vulnerability Correlation | Security Analyst | Correlate discovered software versions with NVD CVEs, risk scoring | P1 | REQ-FN-409 |
| **UC-LS-010** | Scan Schedule & Credential Management | Platform Admin | Per-site schedules, credential vault (CyberArk/HashiCorp), rotation, test connectivity | P0 | REQ-FN-410 |

---

### 3.6 nxa_billing (Billing & Contracts)

| Use Case ID | Name | Actor | Description | Priority | Trace |
|------------|------|-------|-------------|----------|-------|
| **UC-BL-001** | Manage Subscription Plans | Product Admin | Plan definitions: tiers, features, limits, pricing (per asset/user/month), trial | P0 | REQ-FN-501 |
| **UC-BL-002** | Purchase/Upgrade Subscription | Tenant Admin | Self-service checkout (Stripe), proration, downgrade scheduling, cancellation | P0 | REQ-FN-502 |
| **UC-BL-003** | View Invoices & Billing History | Tenant Admin | Invoice list, PDF download, line items, tax, payment status, PO matching | P0 | REQ-FN-503 |
| **UC-BL-004** | Configure Cost Allocation Rules | Finance Admin | Allocation models: per-asset, per-user, tiered, custom formula, cost centers | P0 | REQ-FN-504 |
| **UC-BL-005** | Generate Chargeback/Showback Reports | Finance Admin | Tenant/department reports, budget vs actual, variance analysis, export | P1 | REQ-FN-505 |
| **UC-BL-006** | Manage Contracts & Renewals | Procurement | Contract repository: terms, SLAs, obligations, renewal alerts, approval workflow | P1 | REQ-FN-506 |
| **UC-BL-007** | License True-Up Automation | SAM Manager | Compare discovered installations vs entitlements, generate true-up orders | P1 | REQ-FN-507 |
| **UC-BL-008** | Multi-Currency & Tax Support | Finance Admin | Currency conversion (daily rates), tax rules per jurisdiction, VAT/GST | P1 | REQ-FN-508 |
| **UC-BL-009** | Payment Method Management | Tenant Admin | Cards, bank transfer, invoice terms, auto-pay, failed payment retry | P0 | REQ-FN-509 |
| **UC-BL-010** | Financial Dashboard & Forecasting | CFO/Finance | Revenue recognition, MRR/ARR, churn, LTV, CAC, cohort analysis | P2 | REQ-FN-510 |

---

### 3.7 nxa_notifications (Notifications)

| Use Case ID | Name | Actor | Description | Priority | Trace |
|------------|------|-------|-------------|----------|-------|
| **UC-NT-001** | Configure Notification Channels | Tenant Admin | Email, SMS, Slack, Teams, Webhook, Push, In-app; templates, branding | P0 | REQ-FN-601 |
| **UC-NT-002** | Define Notification Rules | Tenant Admin | Event-driven rules: asset state change, compliance finding, sync failure, threshold | P0 | REQ-FN-602 |
| **UC-NT-003** | Manage Notification Preferences | End User | Per-user channel preferences, frequency (immediate/digest), quiet hours | P1 | REQ-FN-603 |
| **UC-NT-004** | View Notification History | Any Authorized | Searchable log: sent/failed/read, delivery receipts, resend capability | P1 | REQ-FN-604 |
| **UC-NT-005** | Broadcast Announcements | Platform Admin | System-wide announcements, maintenance windows, security advisories | P1 | REQ-FN-605 |

---

### 3.8 nxa_audit (Audit & Compliance)

| Use Case ID | Name | Actor | Description | Priority | Trace |
|------------|------|-------|-------------|----------|-------|
| **UC-AU-001** | Define Compliance Framework | Compliance Admin | Framework library: ISO 27001, SOC 2, GDPR, HIPAA, PCI, NIST, CIS; custom frameworks | P0 | REQ-FN-701 |
| **UC-AU-002** | Map Controls to Policies | Compliance Admin | Control-policy mapping, evidence requirements, testing procedures, frequency | P0 | REQ-FN-702 |
| **UC-AU-003** | Run Compliance Assessment | Compliance Analyst | On-demand/scheduled assessment, evidence collection, findings, remediation tracking | P0 | REQ-FN-703 |
| **UC-AU-004** | Generate Audit Evidence Package | Auditor | Immutable package: findings, evidence, attestations, chain of custody, digital signature | P0 | REQ-FN-704 |
| **UC-AU-005** | Manage Risk Register | Risk Owner | Risk identification, scoring (likelihood×impact), treatment, residual risk, review cycle | P1 | REQ-FN-705 |
| **UC-AU-006** | Incident & Breach Management | Security Team | Incident timeline, evidence preservation, notification workflows (GDPR 72h), post-mortem | P0 | REQ-FN-706 |
| **UC-AU-007** | Data Classification & Handling | Data Steward | Classification labels, handling procedures, DLP rules, retention/disposal schedules | P1 | REQ-FN-707 |
| **UC-AU-008** | Vendor Risk Assessment | Procurement | Vendor questionnaires, scoring, continuous monitoring, contract compliance | P2 | REQ-FN-708 |
| **UC-AU-009** | Policy-as-Code Management | Platform Admin | OPA/Rego policy authoring, testing, versioning, deployment, drift detection | P0 | REQ-FN-709 |
| **UC-AU-010** | Compliance Dashboard & Reporting | Executive | Real-time posture, trend, benchmark, board-ready reports, regulatory submissions | P0 | REQ-FN-710 |

---

### 3.9 nxa_core (Core Platform)

| Use Case ID | Name | Actor | Description | Priority | Trace |
|------------|------|-------|-------------|----------|-------|
| **UC-CR-001** | API Gateway Management | Platform Admin | Route configuration, rate limiting, auth, caching, transformation, canary | P0 | REQ-FN-801 |
| **UC-CR-002** | Developer Portal & API Docs | Developer | OpenAPI 3.1 browsing, try-it-out, SDK generation, API key management | P0 | REQ-FN-802 |
| **UC-CR-003** | Feature Flag Management | Product/Eng | Boolean/multivariate flags, targeting rules, gradual rollout, kill switches | P1 | REQ-FN-803 |
| **UC-CR-004** | System Health & Status Page | All Users | Public status page, component health, incident history, scheduled maintenance | P1 | REQ-FN-804 |
| **UC-CR-005** | Audit Log Query & Export | Auditor/Admin | Tamper-proof log search, filter, export (CSV/JSON/CEF), retention policy | P0 | REQ-FN-805 |
| **UC-CR-006** | Background Job Management | Platform Admin | Job scheduling, monitoring, retry, DLQ, priority queues, cron/calendar | P1 | REQ-FN-806 |

---

## 4. Governance & Compliance Framework

### 4.1 Regulatory & Standards Coverage Matrix

| Framework | Standard | Version | Scope | NexusAsset Coverage | Evidence Artifacts | Assessment Frequency |
|-----------|----------|---------|-------|---------------------|-------------------|---------------------|
| **ISO 27001** | Information Security | 2022 | ISMS | Annex A controls 100% mapped to policies (CAP-031) | Policy docs, risk register, audit logs, evidence packages | Annual (surveillance) |
| **SOC 2** | Trust Services Criteria | 2017 | Security, Availability, Confidentiality | Type II ready: CC1–CC9, A1, C1 | Audit reports, penetration tests, incident records | Annual |
| **GDPR** | Data Protection | 2016 | Personal Data | Art. 5–32: consent, DSAR (UC-TN-006), breach 72h (UC-AU-006), DPA | DPA, RoPA, DPIA, breach log | Continuous |
| **HIPAA** | Health Information | 1996/2013 | PHI | BAAs, encryption, access control, audit, breach notification | BAA, risk analysis, audit logs | Annual |
| **PCI-DSS** | Payment Card | 4.0 | Cardholder Data | Scope reduction (tokenization), network segmentation, logging, ASV scans | AoC, ROC, scan reports | Annual + Quarterly |
| **NIST CSF** | Cybersecurity Framework | 2.0 | Cyber Risk | Identify, Protect, Detect, Respond, Recover mapped to capabilities | Profile, tier assessment, action plan | Annual |
| **CIS Controls** | Critical Security Controls | v8 | Foundational | IG1/IG2/IG3 mapped to platform hardening, vulnerability mgmt | Benchmark reports, remediation tracking | Continuous |

> **Note** — Coverage is **target state** (DOC-01 Section 3.2). Actual certification status tracked in `project-meta/compliance/certification-status.yaml`.

---

### 4.2 Policy Hierarchy & Enforcement

```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                              POLICY HIERARCHY (OPA/Rego)                                    │
├─────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  PLATFORM POLICIES (Global, Immutable)                                              │   │
│  │  • Tenant isolation (RLS enforcement)                                               │   │
│  │  • Encryption at rest / in transit                                                  │   │
│  │  • Audit log immutability                                                           │   │
│  │  • Resource quotas (platform defaults)                                              │   │
│  └─────────────────────────────────────────────────────────────────────────────────────┘   │
│                                      │                                                      │
│                                      ▼                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  TENANT POLICIES (Tenant Admin Configurable)                                        │   │
│  │  • Asset naming conventions                                                         │   │
│  │  • Required tags / custom fields                                                    │   │
│  │  • Approval workflows (asset disposal, high-value procurement)                      │   │
│  │  • Data retention schedules                                                         │   │
│  │  • Integration-specific rules (Lansweeper field ownership)                          │   │
│  └─────────────────────────────────────────────────────────────────────────────────────┘   │
│                                      │                                                      │
│                                      ▼                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  COMPLIANCE POLICIES (Framework-Derived)                                            │   │
│  │  • ISO 27001 A.8.1.1 (Asset Inventory) → CAP-017 policy                             │   │
│  │  • SOC 2 CC6.1 (Logical Access) → CAP-052 RBAC/ABAC policy                          │   │
│  │  • GDPR Art. 25 (Data Protection by Design) → CAP-035 classification policy        │   │
│  │  • CIS 4.1 (Secure Config) → CAP-031 hardening policy                               │   │
│  └─────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                             │
│  ENFORCEMENT: OPA Sidecar (Envoy) → Decision Logs → Loki → Grafana Alerting               │
│                                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

### 4.3 Data Governance & Classification

| Classification Level | Label | Description | Handling Requirements | Retention | Encryption |
|---------------------|-------|-------------|----------------------|-----------|------------|
| **Public** | `PUBLIC` | Marketing, public docs, status page | No restrictions | Indefinite | TLS only |
| **Internal** | `INTERNAL` | Internal ops, non-sensitive configs | Employee access only | 7 years | AES-256 at rest |
| **Confidential** | `CONFIDENTIAL` | Customer data, financials, contracts | Need-to-know, MFA, audit | 7 years + legal hold | AES-256 + per-tenant DEK |
| **Restricted** | `RESTRICTED` | PII, PHI, secrets, keys, audit evidence | Strict RBAC, encryption, DLP, SIEM | Legal requirement | AES-256 + per-tenant DEK + HSM |

**Data Classification Rules** (BR-DG-001 to BR-DG-012):
- **BR-DG-001**: All tenant data defaults to `CONFIDENTIAL`
- **BR-DG-002**: PII/PHI fields (email, phone, SSN, medical record #) → `RESTRICTED`
- **BR-DG-003**: Encryption keys, API secrets, certificates → `RESTRICTED`
- **BR-DG-004**: Audit logs, evidence packages → `RESTRICTED` (immutable)
- **BR-DG-005**: Discovery scan raw artifacts → `CONFIDENTIAL` (90 days) → `INTERNAL` (archival)
- **BR-DG-006**: Cross-tenant queries prohibited at database level (RLS enforcement)
- **BR-DG-007**: Data export requires `CONFIDENTIAL` handling, watermarking, expiry
- **BR-DG-008**: Backup encryption uses tenant-specific DEK wrapped by platform KEK
- **BR-DG-009**: Data residency: EU tenants → EU regions only (enforced by provisioning)
- **BR-DG-010**: Retention policies enforced by automated lifecycle jobs (CAP-084)

---

## 5. Audit Requirements & Evidence Generation

### 5.1 Audit Event Taxonomy

Every auditable action emits a structured **Audit Event** (EVT-AU-###) to the immutable audit log (append-only, cryptographically chained).

| Event Category | Event IDs | Description | Retention |
|----------------|-----------|-------------|-----------|
| **Authentication** | EVT-AU-001 to 010 | Login, MFA, SSO, token refresh, impersonation start/end | 7 years |
| **Authorization** | EVT-AU-011 to 020 | Role assignment, policy change, permission check (deny), delegation | 7 years |
| **Asset Lifecycle** | EVT-AU-021 to 040 | Create, update, state change, assign, relocate, retire, dispose | 7 years |
| **Discovery & Sync** | EVT-AU-041 to 050 | Scan start/end, raw artifact stored, reconciliation decisions, conflicts | 3 years |
| **Tenant Operations** | EVT-AU-051 to 060 | Provision, config change, suspend, export, offboard, domain change | 7 years |
| **Billing & Financial** | EVT-AU-061 to 070 | Invoice, payment, subscription change, allocation rule change | 7 years + tax law |
| **Compliance & Policy** | EVT-AU-071 to 090 | Policy eval, finding, remediation, evidence package, risk update | 7 years |
| **Security Events** | EVT-AU-091 to 100 | Vuln scan, secret rotation, cert renewal, breach, DLP alert | 7 years |
| **Platform Operations** | EVT-AU-101 to 110 | Deploy, rollback, config drift, backup, DR test, chaos experiment | 3 years |

---

### 5.2 Evidence Package Specification

**Trigger**: Scheduled (compliance calendar), on-demand (auditor request), incident-driven

**Package Contents** (signed JWS with platform signing key):

```json
{
  "packageId": "EVT-AU-PKG-2026-001234",
  "tenantId": "tn_abc123",
  "framework": "ISO_27001_2022",
  "controlId": "A.8.1.1",
  "period": { "start": "2026-01-01", "end": "2026-03-31" },
  "generatedAt": "2026-04-01T12:00:00Z",
  "generatedBy": "sys_audit_engine",
  "evidence": [
    { "type": "policy_document", "ref": "POL-031-asset-inventory", "hash": "sha256:..." },
    { "type": "asset_inventory_snapshot", "ref": "s3://audit-evidence/tn_abc123/assets-2026-Q1.json.gz", "count": 15420, "hash": "sha256:..." },
    { "type": "reconciliation_log", "ref": "s3://audit-evidence/tn_abc123/recon-2026-Q1.log.gz", "hash": "sha256:..." },
    { "type": "access_control_matrix", "ref": "s3://audit-evidence/tn_abc123/rbac-2026-Q1.json", "hash": "sha256:..." },
    { "type": "audit_log_segment", "ref": "s3://audit-evidence/tn_abc123/audit-2026-Q1.log.gz", "hash": "sha256:..." }
  ],
  "attestation": {
    "attestor": "sys_audit_engine",
    "algorithm": "RS256",
    "signature": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
  },
  "chainOfCustody": [
    { "handler": "sys_audit_engine", "action": "generated", "timestamp": "2026-04-01T12:00:00Z" },
    { "handler": "auditor_jdoe", "action": "downloaded", "timestamp": "2026-04-02T09:15:00Z" }
  ]
}
```

---

### 5.3 Immutable Audit Log Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                          AUDIT LOG ARCHITECTURE                                              │
├─────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                             │
│  APPLICATION SERVICES                                                                        │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                                                       │
│  │ nxa_    │  │ nxa_    │  │ nxa_    │  ... (all services)                                  │
│  │ assets  │  │ accounts│  │ audit   │                                                       │
│  └────┬────┘  └────┬────┘  └────┬────┘                                                       │
│       │            │            │                                                             │
│       └────────────┼────────────┘                                                             │
│                    ▼                                                                          │
│  ┌─────────────────────────────────────────┐                                                │
│  │         AUDIT INTERCEPTOR (Middleware)  │  ← Structured logging, correlation IDs         │
│  │  • Enrich with tenant, user, request ID │                                                │
│  │  • Add timestamp (RFC3339, monotonic)   │                                                │
│  │  • Serialize to JSON Lines (NDJSON)     │                                                │
│  └────────────────────┬────────────────────┘                                                │
│                       │                                                                       │
│                       ▼                                                                       │
│  ┌─────────────────────────────────────────┐                                                │
│  │         KAFKA TOPIC: nxa.audit.raw      │  ← Partitioned by tenant_id                    │
│  │  • Retention: 90 days (hot)             │                                                │
│  │  • Compression: ZSTD                    │                                                │
│  └────────────────────┬────────────────────┘                                                │
│                       │                                                                       │
│           ┌───────────┴───────────┐                                                           │
│           ▼                       ▼                                                           │
│  ┌──────────────────┐    ┌──────────────────┐                                                │
│  │  HOT PATH        │    │  COLD PATH       │                                                │
│  │  (Real-time)     │    │  (Archival)      │                                                │
│  ├──────────────────┤    ├──────────────────┤                                                │
│  │ • Loki (logs)    │    │ • S3/MinIO       │                                                │
│  │ • Grafana Alerts │    │   (Parquet)      │                                                │
│  │ • SIEM Forward   │    │ • Glacier (7yr)  │                                                │
│  │ • Stream Proc    │    │ • Merkle Tree    │                                                │
│  └──────────────────┘    └──────────────────┘                                                │
│           │                       │                                                           │
│           └───────────┬───────────┘                                                           │
│                       ▼                                                                       │
│  ┌─────────────────────────────────────────┐                                                │
│  │         VERIFICATION LAYER              │                                                │
│  │  • Merkle root published to transparency│                                                │
│  │    log (CT-style) hourly                │                                                │
│  │  • Inclusion proofs for any event       │                                                │
│  │  • Tamper-evident: any modification     │                                                │
│  │    breaks chain → alert                 │                                                │
│  └─────────────────────────────────────────┘                                                │
│                                                                                             │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Key Performance Indicators (KPIs) & Success Metrics

### 6.1 Strategic KPIs (Executive Level)

| KPI ID | Name | Formula | Target | Frequency | Owner | Trace |
|--------|------|---------|--------|-----------|-------|-------|
| **KPI-001** | Asset Coverage Ratio | (Managed Assets / Total Discoverable Assets) × 100 | ≥ 95% | Monthly | VP Product | CAP-010 |
| **KPI-002** | Data Accuracy Index | 1 - (Confirmed Errors / Total Attributes) | ≥ 99.5% | Weekly | Data Steward | CAP-016 |
| **KPI-003** | Compliance Posture Score | Weighted avg of framework control pass rates | ≥ 95% | Continuous | CISO | CAP-030 |
| **KPI-004** | Tenant Activation Rate | (Active Tenants / Provisioned Tenants) × 100 | ≥ 80% | Monthly | VP Sales | CAP-050 |
| **KPI-005** | Net Revenue Retention (NRR) | (Starting MRR + Expansion - Contraction - Churn) / Starting MRR | ≥ 110% | Quarterly | CFO | CAP-040 |
| **KPI-006** | Mean Time to Remediate (MTTR) | Avg time from finding creation to closure (critical) | < 24 hours | Continuous | Security Lead | CAP-034 |
| **KPI-007** | Platform Availability | Uptime % (excluding planned maintenance) | 99.95% | Monthly | SRE Lead | CAP-080 |
| **KPI-008** | Discovery Freshness | Max age of asset data across all sources | < 4 hours (P0), < 24h (P1) | Continuous | Platform Eng | CAP-010 |

---

### 6.2 Operational KPIs (Engineering Level)

| KPI ID | Name | Formula | Target | Frequency | Owner |
|--------|------|---------|--------|-----------|-------|
| **KPI-101** | Sync Success Rate | (Successful Syncs / Total Syncs) × 100 | ≥ 99.5% | Per sync | Platform Eng |
| **KPI-102** | Sync Latency (P95) | Time from source change to CMDB update | < 5 min (webhook), < 1 hr (scheduled) | Continuous | Platform Eng |
| **KPI-103** | API Latency (P99) | p99 of /api/v1/assets/* endpoints | < 500 ms | Continuous | Backend Lead |
| **KPI-104** | API Error Rate | 5xx / Total requests | < 0.1% | Continuous | Backend Lead |
| **KPI-105** | Reconciliation Conflict Rate | Conflicts requiring human review / Total reconciled | < 2% | Daily | Data Steward |
| **KPI-106** | Background Job Queue Depth | Pending jobs > 5 min old | < 100 | Continuous | Platform Eng |
| **KPI-107** | Database Query Performance | p95 query time (tenant-scoped) | < 100 ms | Continuous | DBA |
| **KPI-108** | Cache Hit Ratio | Redis cache hits / (hits + misses) | > 95% | Continuous | Backend Lead |

---

### 6.3 Quality & Security KPIs

| KPI ID | Name | Formula | Target | Frequency | Owner |
|--------|------|---------|--------|-----------|-------|
| **KPI-201** | Vulnerability Remediation SLA | % critical vulns patched within 72h | 100% | Continuous | SecOps |
| **KPI-202** | Secret Rotation Compliance | % secrets rotated within policy window | 100% | Monthly | SecOps |
| **KPI-203** | Test Coverage (Domain) | Line/branch coverage in domain layer | ≥ 90% | Per PR | QA Lead |
| **KPI-204** | Contract Test Pass Rate | Consumer-driven contract tests passing | 100% | Per deploy | QA Lead |
| **KPI-205** | Chaos Experiment Success | % experiments completing without SLO breach | 100% | Monthly | SRE Lead |
| **KPI-206** | Backup Restore Test | Successful restore / Scheduled tests | 100% | Quarterly | SRE Lead |

---

## 7. Stakeholder Value Mapping

### 7.1 Stakeholder Register

| Stakeholder ID | Role | Organization | Interest | Influence | Communication Cadence |
|----------------|------|--------------|----------|-----------|----------------------|
| **SH-001** | CIO / CTO | Customer | Strategic alignment, ROI, security | High | Monthly steering |
| **SH-002** | IT Asset Manager | Customer | Daily operations, accuracy, efficiency | High | Weekly sync |
| **SH-003** | Security/Compliance Officer | Customer | Audit readiness, policy enforcement | High | Bi-weekly |
| **SH-004** | Finance/Procurement | Customer | Cost control, license optimization | Medium | Monthly |
| **SH-005** | End User (Employee) | Customer | Self-service, asset request, transparency | Low | Quarterly survey |
| **SH-006** | MSP / Service Provider | Partner | Multi-tenancy, white-label, margins | High | Monthly business review |
| **SH-007** | Auditor (Internal/External) | Independent | Evidence, compliance, controls | High | Per audit cycle |
| **SH-008** | Platform Engineering | NexusAsset | Reliability, scalability, developer velocity | High | Daily standup |
| **SH-009** | Product Management | NexusAsset | Feature adoption, competitive differentiation | High | Sprint review |
| **SH-010** | Sales / Customer Success | NexusAsset | Renewals, expansion, referenceability | High | Weekly pipeline |

---

### 7.2 Value Proposition Matrix

| Capability | SH-001 (CIO) | SH-002 (Asset Mgr) | SH-003 (Sec/Comp) | SH-004 (Finance) | SH-006 (MSP) | SH-007 (Auditor) |
|------------|--------------|-------------------|-------------------|------------------|--------------|------------------|
| **CAP-010 Discovery** | Strategic visibility | Automation, accuracy | Complete inventory | Cloud cost allocation | Scalable scanning | Evidence of coverage |
| **CAP-020 Lifecycle** | Governance, risk reduction | Workflow efficiency | Chain of custody | Asset financials | Service delivery | Lifecycle audit trail |
| **CAP-030 Governance** | Risk posture, board reporting | Policy automation | **Core value** | Control compliance | Policy inheritance | **Primary evidence** |
| **CAP-040 Financial** | ROI, cost optimization | Budget tracking | License compliance | **Core value** | Billing automation | Financial controls |
| **CAP-050 Tenancy** | Multi-org support | Data isolation | **Regulatory req** | Cost allocation | **Core product** | Tenant isolation proof |
| **CAP-060 Integration** | Ecosystem fit | ITSM workflow | SIEM feed | Vendor mgmt | Extensibility | Integration audit |
| **CAP-070 Reporting** | Board dashboards | Operational views | Compliance reports | Financial reports | Customer reports | Audit-ready packages |
| **CAP-080 Platform** | Reliability, scale | Performance | Security ops | Infrastructure cost | SLA delivery | Availability evidence |

---

## 8. Business Rules Catalog

### 8.1 Rule Identification Scheme

| Prefix | Domain | Example |
|--------|--------|---------|
| **BR-DG** | Data Governance | BR-DG-001 |
| **BR-AL** | Asset Lifecycle | BR-AL-001 |
| **BR-TN** | Tenancy | BR-TN-001 |
| **BR-BL** | Billing | BR-BL-001 |
| **BR-CP** | Compliance | BR-CP-001 |
| **BR-LS** | Lansweeper Sync | BR-LS-001 |
| **BR-AC** | Access Control | BR-AC-001 |

---

### 8.2 Core Business Rules (Sample — Full catalog in Appendix)

| Rule ID | Rule Statement | Context | Enforcement | Trace |
|---------|---------------|---------|-------------|-------|
| **BR-AL-001** | An asset MUST have exactly one current owner (user or group) when in `Active` state | Asset Lifecycle | Domain invariant (aggregate) | UC-AS-005 |
| **BR-AL-002** | Asset disposal REQUIRES verified data sanitization certificate (NIST 800-88) | Retirement | Workflow gate + policy | UC-AS-009 |
| **BR-AL-003** | CI relationships MUST form a directed acyclic graph (no circular dependencies) | CMDB | Validation on write | UC-AS-010 |
| **BR-TN-001** | Tenant data MUST be isolated at database level via RLS (no cross-tenant queries) | Multi-tenancy | DB policy + integration test | UC-TN-001 |
| **BR-TN-002** | Tenant offboarding REQUIRES 30-day grace period unless legal hold | Tenant Lifecycle | Workflow + scheduler | UC-TN-008 |
| **BR-TN-003** | Custom domains MUST pass CNAME verification + TLS provisioning before activation | Branding | Async verification job | UC-TN-009 |
| **BR-BL-001** | Invoice generation is IDEMPOTENT per billing period per tenant | Billing | Idempotency key (tenant+period) | UC-BL-003 |
| **BR-BL-002** | Cost allocation rules MUST sum to 100% for each allocation dimension | Chargeback | Validation on save | UC-BL-004 |
| **BR-CP-001** | Critical compliance findings (CVSS ≥ 9.0) MUST trigger incident within 15 min | Incident Mgmt | Policy engine + alerting | UC-AU-006 |
| **BR-CP-002** | Audit evidence packages MUST be cryptographically signed and tamper-evident | Evidence | JWS + Merkle tree | UC-AU-004 |
| **BR-LS-001** | Lansweeper is SYSTEM OF RECORD for hardware attributes (CPU, RAM, disk, MAC) | Field Ownership | Reconciliation engine | UC-LS-004 |
| **BR-LS-002** | Cloud provider API is SYSTEM OF RECORD for cloud tags, region, instance type | Field Ownership | Reconciliation engine | UC-LS-004 |
| **BR-LS-003** | Manual entry is SYSTEM OF RECORD for owner, location, cost center, custom fields | Field Ownership | Reconciliation engine | UC-LS-004 |
| **BR-AC-001** | Tenant admin CANNOT assign platform-level roles (superadmin, platform-operator) | RBAC | Policy engine (OPA) | UC-AC-005 |
| **BR-AC-002** | Impersonation sessions MAX 60 min, require approval, full audit trail | Support | Workflow + audit log | UC-AC-007 |
| **BR-DG-001** | All tenant data defaults to CONFIDENTIAL classification | Data Gov | Default on create | UC-TN-001 |

> **Note** — Complete business rules catalog (150+ rules) maintained in `docs/04-domain/02-business-rules.yaml` with machine-readable format for policy engine compilation.

---

## 9. Risk & Assumption Register (Business View)

### 9.1 Business Risks

| Risk ID | Risk Statement | Likelihood | Impact | Score | Mitigation | Owner | Status |
|---------|----------------|------------|--------|-------|------------|-------|--------|
| **RSK-B-001** | Lansweeper API changes break sync (vendor lock-in) | Medium | High | 12 | Abstraction layer, contract tests, multi-source reconciliation | Platform Lead | Active |
| **RSK-B-002** | Tenant data leakage due to RLS misconfiguration | Low | Critical | 10 | Automated RLS verification tests, penetration testing, chaos engineering | Security Lead | Active |
| **RSK-B-003** | Regulatory changes require rapid policy updates (GDPR, AI Act) | Medium | High | 12 | Policy-as-code, automated compliance mapping, legal review cycle | Compliance Lead | Active |
| **RSK-B-004** | Large tenant onboarding causes platform performance degradation | Medium | Medium | 9 | Load testing, auto-scaling, tenant isolation benchmarks | SRE Lead | Active |
| **RSK-B-005** | Bill shock / unexpected charges damage trust | Low | High | 8 | Real-time usage alerts, budget caps, transparent chargeback | Product Lead | Active |
| **RSK-B-006** | Competitor launches AI-powered predictive ITAM | Medium | Medium | 9 | ML roadmap (CAP-074), partnership strategy | VP Product | Monitoring |
| **RSK-B-007** | Key personnel loss (domain experts, security) | Low | High | 8 | Documentation, cross-training, knowledge management | Engineering Lead | Active |
| **RSK-B-008** | Supply chain vulnerability in dependencies (Log4j-style) | Medium | Critical | 15 | SBOM, dependency scanning, rapid patch pipeline | Security Lead | Active |
| **RSK-B-009** | Data residency requirements force multi-region complexity | Medium | High | 12 | Region-aware provisioning, automated failover testing | Platform Lead | Planned |
| **RSK-B-010** | Audit evidence challenged as insufficient/tampered | Low | Critical | 10 | Merkle tree, transparency log, third-party attestation | Compliance Lead | Active |

---

### 9.2 Business Assumptions

| Assumption ID | Assumption | Validation Method | Risk if False |
|---------------|------------|-------------------|---------------|
| **ASM-B-001** | Lansweeper remains primary on-prem discovery source for 3+ years | Vendor roadmap review, contract term | RSK-B-001 |
| **ASM-B-002** | PostgreSQL 18 RLS performance scales to 10M assets/tenant | Load testing (DOC-06), benchmarking | RSK-B-004 |
| **ASM-B-003** | Customers accept eventual consistency (< 5 min) for discovery data | User research, SLA definition | RSK-B-005 |
| **ASM-B-004** | OpenAPI 3.1 + JWT is sufficient for 95% of integration needs | Market analysis, PoC with top 10 prospects | CAP-064 |
| **ASM-B-005** | MSPs require white-label / hierarchical tenancy | Sales feedback, competitive analysis | CAP-050, UC-TN-010 |
| **ASM-B-006** | Policy-as-code (OPA) adoption reduces compliance cost by 40% | Pilot with design partners | RSK-B-003 |
| **ASM-B-007** | Kubernetes (EKS/GKE/AKS) is standard deployment target | Customer survey, cloud spend analysis | DOC-06 |
| **ASM-B-008** | Webhook-based delta sync reduces full scan frequency by 80% | Lansweeper webhook reliability testing | CAP-011 |

---

## 10. Traceability Matrix

### 10.1 Capability → Use Case → Requirement → Test Case

| Capability | Use Cases | Functional Requirements | Non-Functional Requirements | Test Cases |
|------------|-----------|------------------------|----------------------------|------------|
| **CAP-011** | UC-LS-001, UC-LS-002, UC-LS-003, UC-LS-007, UC-LS-010 | REQ-FN-401..410 | REQ-NFR-001 (performance), REQ-NFR-010 (security) | TC-INT-001..010, TC-E2E-001 |
| **CAP-016** | UC-AS-001, UC-AS-002, UC-AS-011, UC-LS-004, UC-LS-005 | REQ-FN-301, 302, 311, 404, 405 | REQ-NFR-002 (accuracy), REQ-NFR-003 (consistency) | TC-UNIT-050..060, TC-INT-020 |
| **CAP-017** | UC-AS-003, UC-AS-010 | REQ-FN-303, 310 | REQ-NFR-004 (graph query perf) | TC-UNIT-070, TC-E2E-010 |
| **CAP-018** | UC-AS-005, UC-AS-006, UC-AS-009 | REQ-FN-305, 306, 309 | REQ-NFR-005 (state machine integrity) | TC-UNIT-080..090 |
| **CAP-031** | UC-AU-001, UC-AU-002, UC-AU-003, UC-AU-009 | REQ-FN-701, 702, 703, 709 | REQ-NFR-011 (policy eval latency), REQ-NFR-012 (audit) | TC-UNIT-100..110, TC-CHAOS-001 |
| **CAP-033** | UC-AU-004, UC-AU-010 | REQ-FN-704, 710 | REQ-NFR-013 (immutability), REQ-NFR-014 (sig verification) | TC-UNIT-120, TC-SEC-001 |
| **CAP-051** | UC-TN-001, UC-TN-002, UC-TN-009, UC-TN-010 | REQ-FN-201, 202, 209, 210 | REQ-NFR-015 (provisioning time), REQ-NFR-016 (isolation) | TC-INT-030..040, TC-SEC-010 |
| **CAP-053** | UC-TN-001, UC-TN-006, UC-TN-008 | REQ-FN-201, 206, 208 | REQ-NFR-017 (RLS enforcement), REQ-NFR-018 (encryption) | TC-SEC-020..030, TC-PERF-001 |
| **CAP-061** | UC-LS-001..010 | REQ-FN-401..410 | REQ-NFR-019 (sync reliability), REQ-NFR-020 (idempotency) | TC-INT-001..050, TC-CHAOS-002 |
| **CAP-071** | UC-AS-003, UC-AS-004, UC-LS-003 | REQ-FN-303, 304, 403 | REQ-NFR-021 (dashboard latency), REQ-NFR-022 (real-time) | TC-E2E-020, TC-PERF-010 |

> **Complete traceability matrix** (all 80 capabilities × 90 use cases × 200+ requirements × 500+ test cases) maintained in `docs/14-appendices/traceability-matrix.xlsx` and auto-generated in CI.

---

## 11. Diagrams

| Diagram ID | Title | Type | Location | Description |
|------------|-------|------|----------|-------------|
| **DIA-001** | Capability Map (Level 1) | Hierarchy | Section 1.1 | Strategic capability taxonomy |
| **DIA-002** | Core Business Process Map | BPMN | Section 2.1 | End-to-end process flow |
| **DIA-003** | Asset Lifecycle State Machine | State Diagram | Section 2.5 | 6-state lifecycle with transitions |
| **DIA-004** | Policy Hierarchy & Enforcement | Layered | Section 4.2 | Platform → Tenant → Compliance policies |
| **DIA-005** | Audit Log Architecture | Component | Section 5.3 | Hot/cold paths, Merkle tree, verification |
| **DIA-006** | Stakeholder Value Heatmap | Matrix | Section 7.2 | Capability × Stakeholder value |
| **DIA-007** | Field Ownership Matrix | Table/Matrix | Section 8.2 (BR-LS-001..003) | Source priority per attribute |
| **DIA-008** | Risk Heat Map | Scatter | Section 9.1 | Likelihood × Impact for business risks |
| **DIA-009** | Reconciliation Data Flow | Sequence | Appendix A | Event flow: Discovered → Reconciled → Authorized |
| **DIA-010** | Tenant Provisioning Sequence | Sequence | Appendix B | Self-service → Schema → RLS → Branding → Ready |

---

## 12. Tables

| Table ID | Title | Location | Row Count |
|----------|-------|----------|-----------|
| **TBL-001** | Level 1 Capability Map | Section 1.1 | 8 |
| **TBL-002** | CAP-010 Decomposition | Section 1.2 | 8 |
| **TBL-003** | CAP-020 Decomposition | Section 1.3 | 7 |
| **TBL-004** | CAP-030 Decomposition | Section 1.4 | 7 |
| **TBL-005** | CAP-040 Decomposition | Section 1.5 | 5 |
| **TBL-006** | CAP-050 Decomposition | Section 1.6 | 5 |
| **TBL-007** | CAP-060 Decomposition | Section 1.7 | 5 |
| **TBL-008** | CAP-070 Decomposition | Section 1.8 | 4 |
| **TBL-009** | CAP-080 Decomposition | Section 1.9 | 5 |
| **TBL-010** | PROC-010 Activities | Section 2.2 | 6 |
| **TBL-011** | PROC-020 Activities | Section 2.3 | 6 |
| **TBL-012** | PROC-030 Activities | Section 2.4 | 6 |
| **TBL-013** | PROC-040 State Transitions | Section 2.5 | 6 |
| **TBL-014** | Regulatory Coverage Matrix | Section 4.1 | 7 |
| **TBL-015** | Data Classification Levels | Section 4.3 | 4 |
| **TBL-016** | Audit Event Taxonomy | Section 5.1 | 9 |
| **TBL-017** | Strategic KPIs | Section 6.1 | 8 |
| **TBL-018** | Operational KPIs | Section 6.2 | 8 |
| **TBL-019** | Quality & Security KPIs | Section 6.3 | 6 |
| **TBL-020** | Stakeholder Register | Section 7.1 | 10 |
| **TBL-021** | Stakeholder Value Matrix | Section 7.2 | 8×6 |
| **TBL-022** | Business Rules (Sample) | Section 8.2 | 18 |
| **TBL-023** | Business Risks | Section 9.1 | 10 |
| **TBL-024** | Business Assumptions | Section 9.2 | 8 |
| **TBL-025** | Capability Traceability Matrix | Section 10.1 | 10 |
| **TBL-026** | Diagram Index | Section 11 | 10 |
| **TBL-027** | Table Index | Section 12 | 27 |

---

---

**End of DOC-02 — NexusAsset Cloud Business Analysis**

*Document Control: Version 1.0.0 | Classification: Internal — Confidential | Next Review: 2026-12-09*

*For questions or updates: docs@nexusasset.cloud | Architecture Review Board: arch-review@nexusasset.cloud*