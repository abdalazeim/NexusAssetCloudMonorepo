┌──────────────────────────────────────────────────────────────┐
│  File        : 02-design/05-data-model.md                   │
│  Project     : NexusAsset Cloud                              │
│  Module      : 02-design                                    │
│  Version     : 1.0.0                                         │
│  Status      : Draft                                        │
│  Owner       : Architecture                                 │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  DATA MODEL                                                    ║
║  ERD, Multi-Tenancy, Indexes, and Retention                    ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Purpose](#1-purpose)
2. [Scope](#2-scope)
3. [Standard](#3-standard)
4. [Implementation Guidance](#4-implementation-guidance)
5. [Validation](#5-validation)
6. [Related Files](#6-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Purpose                                               │
└──────────────────────────────────────────────────────────┘

Define the NexusAsset Cloud standard for erd, multi-tenancy, indexes, and retention.

┌──────────────────────────────────────────────────────────┐
│  2. Scope                                                 │
└──────────────────────────────────────────────────────────┘

This document applies to the NexusAsset Cloud monorepo, delivery pipelines, and runtime environments unless a more specific approved document states otherwise.

┌──────────────────────────────────────────────────────────┐
│  3. Standard                                              │
└──────────────────────────────────────────────────────────┘

Use NexusAsset Cloud canonical naming: repository `nexusasset-cloud/`, Django package `backend/nexusasset/`, apps `nxa_*`, services `nxa-backend/nxa-frontend/nxa-worker/nxa-db/nxa-redis`, edge `nxa-traefik-public` and `nxa-le`, domains `app.nexusasset.cloud` and `api.nexusasset.cloud`, packages `@nexusasset/*`, and secrets `NXA_*`.

Enforce Clean Architecture, tenant isolation (`tenant_id` + PostgreSQL RLS), idempotent integrations, and OpenAPI 3.1 as the contract source of truth.

┌──────────────────────────────────────────────────────────┐
│  4. Implementation Guidance                               │
└──────────────────────────────────────────────────────────┘

▸ Requirements
- Use active, testable, and reviewable controls for data model.
- Align implementation with bounded contexts (`nxa_core`, `nxa_accounts`, `nxa_tenants`, `nxa_assets`, `nxa_lansweeper`, `nxa_billing`, `nxa_notifications`, `nxa_audit`).
- Document exceptions through Architecture Decision Records (ADRs).

╭─ NOTE ──────────────────────────────────────────────────────╮
│ This document is a canonical NexusAsset Cloud standard.     │
│ Update it through pull request review and ADR linkage when  │
│ architecture-impacting changes occur.                       │
╰─────────────────────────────────────────────────────────────╯

┌──────────────────────────────────────────────────────────┐
│  5. Validation                                            │
└──────────────────────────────────────────────────────────┘

| Check | Expected Result |
|---|---|
| Naming | Uses canonical NexusAsset Cloud names |
| Security | Protects secrets, tenant boundaries, and access controls |
| Quality | Passes lint, type, architecture, and test gates |
| Traceability | Links to related docs, ADRs, and tests |

┌──────────────────────────────────────────────────────────┐
│  6. Related Files                                         │
└──────────────────────────────────────────────────────────┘

- [../01-analysis/08-bounded-contexts.md](../01-analysis/08-bounded-contexts.md)
- [../03-implementation/01-project-structure.md](../03-implementation/01-project-structure.md)


┌──────────────────────────────────────────────────────────┐
│  Adapted Source: 02_Architecture_and_Design/04_Database_Design_Principles.txt│
└──────────────────────────────────────────────────────────┘
▸ Distributed from Fundamentals reference library (rewritten for NexusAsset Cloud)
- ═══════════════════════════════════════════════════════
- ───────────────────────────────────────────────────────
- ─────────────────────────────────────────────────

╭─ NOTE ──────────────────────────────────────────────────────╮
│ Apply these practices with NexusAsset Cloud standards:      │
│ Clean Architecture, `nxa_*` contexts, OpenAPI 3.1,          │
│ `tenant_id` + PostgreSQL RLS, and security quality gates.   │
╰─────────────────────────────────────────────────────────────╯
─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: docs@nexusasset.cloud
─────────────────────────────────────────────────────────────
