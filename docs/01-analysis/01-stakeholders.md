┌──────────────────────────────────────────────────────────────┐
│  File        : 01-analysis/01-stakeholders.md               │
│  Project     : NexusAsset Cloud                              │
│  Module      : 01-analysis                                  │
│  Version     : 1.0.0                                         │
│  Status      : Draft                                        │
│  Owner       : Product & Architecture                       │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  STAKEHOLDERS                                                  ║
║  Roles, Responsibilities, and Decision Rights                  ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Purpose](#1-purpose)
2. [Stakeholder Matrix](#2-stakeholder-matrix)
3. [Decision Rights](#3-decision-rights)
4. [Communication](#4-communication)
5. [Standards](#5-standards)
6. [Related Files](#6-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Purpose                                               │
└──────────────────────────────────────────────────────────┘

Identify stakeholders who influence NexusAsset Cloud scope, delivery, security, and operations.

┌──────────────────────────────────────────────────────────┐
│  2. Stakeholder Matrix                                    │
└──────────────────────────────────────────────────────────┘

| Stakeholder | Interest | Responsibility | Engagement |
|---|---|---|---|
| Product Owner | Value and roadmap | Prioritization, acceptance | Sprint planning/review |
| Platform Architect | Technical integrity | Architecture decisions, ADRs | Design reviews |
| Backend Team | API/domain delivery | Django/DRF implementation | Daily delivery |
| Frontend Team | Web UX/BFF | Next.js implementation | Daily delivery |
| DevOps/SRE | Reliability/deployment | CI/CD, infra, observability | Release/ops |
| Security/Compliance | Risk and controls | Threat model, scans, audits | Gates/audits |
| Tenant Admins | Tenant outcomes | Configuration and user management | UAT/feedback |

┌──────────────────────────────────────────────────────────┐
│  3. Decision Rights                                       │
└──────────────────────────────────────────────────────────┘

Architecture changes require ADRs. Security, tenant isolation, and quality gates are non-negotiable release criteria.

┌──────────────────────────────────────────────────────────┐
│  4. Communication                                         │
└──────────────────────────────────────────────────────────┘

Use roadmap reviews, sprint ceremonies, pull requests, ADRs, and incident/runbook updates.

┌──────────────────────────────────────────────────────────┐
│  5. Standards                                             │
└──────────────────────────────────────────────────────────┘

Use canonical NexusAsset Cloud naming (`nxa-*`, `@nexusasset/*`, `NXA_*`, `*.nexusasset.cloud`). Enforce Clean Architecture, tenant isolation (`tenant_id` + PostgreSQL RLS), and OpenAPI-first contracts.

┌──────────────────────────────────────────────────────────┐
│  6. Related Files                                         │
└──────────────────────────────────────────────────────────┘

- [./02-personas.md](./02-personas.md)
- [../11-governance/05-team-structure.md](../11-governance/05-team-structure.md)


┌──────────────────────────────────────────────────────────┐
│  Adapted Source: 01_Planning_and_Analysis/01_Planning.txt│
└──────────────────────────────────────────────────────────┘
▸ Distributed from Fundamentals reference library (rewritten for NexusAsset Cloud)
- ═══════════════════════════════════════════════════════════════════
- ───────────────────────────────────────────────────────────────────
- ┌──────────────────────┬──────────────┬──────────────┬──────────────┐
- ├──────────────────────┼──────────────┼──────────────┼──────────────┤

╭─ NOTE ──────────────────────────────────────────────────────╮
│ Apply these practices with NexusAsset Cloud standards:      │
│ Clean Architecture, `nxa_*` contexts, OpenAPI 3.1,          │
│ `tenant_id` + PostgreSQL RLS, and security quality gates.   │
╰─────────────────────────────────────────────────────────────╯


┌──────────────────────────────────────────────────────────┐
└──────────────────────────────────────────────────────────┘
▸ Distributed from Fundamentals reference library (rewritten for NexusAsset Cloud)
- ═══════════════════════════════════════════════════════
- ───────────────────────────────────────────────────────
- ┌─────────────────────┬──────────────────────────────────┐
- ├─────────────────────┼──────────────────────────────────┤

╭─ NOTE ──────────────────────────────────────────────────────╮
│ Apply these practices with NexusAsset Cloud standards:      │
│ Clean Architecture, `nxa_*` contexts, OpenAPI 3.1,          │
│ `tenant_id` + PostgreSQL RLS, and security quality gates.   │
╰─────────────────────────────────────────────────────────────╯
─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: docs@nexusasset.cloud
─────────────────────────────────────────────────────────────
