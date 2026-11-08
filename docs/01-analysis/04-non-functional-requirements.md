┌──────────────────────────────────────────────────────────────┐
│  File        : 01-analysis/04-non-functional-requirements.md│
│  Project     : NexusAsset Cloud                              │
│  Module      : 01-analysis                                  │
│  Version     : 1.0.0                                         │
│  Status      : Draft                                        │
│  Owner       : Architecture & SRE                           │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  NON-FUNCTIONAL REQUIREMENTS                                   ║
║  Quality Attributes and Targets                                ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Purpose](#1-purpose)
2. [NFR Matrix](#2-nfr-matrix)
3. [Constraints](#3-constraints)
4. [Verification](#4-verification)
5. [Standards](#5-standards)
6. [Related Files](#6-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Purpose                                               │
└──────────────────────────────────────────────────────────┘

Define measurable quality attributes for NexusAsset Cloud.

┌──────────────────────────────────────────────────────────┐
│  2. NFR Matrix                                            │
└──────────────────────────────────────────────────────────┘

| Category | Target | Mechanism |
|---|---|---|
| Performance | p95 read < 400 ms; p95 write < 800 ms | Caching, indexes, async jobs, load tests |
| Availability | 99.9% core services | Health checks, restart policies, replication |
| Scalability | Horizontal API/workers | Stateless `nxa-backend`, Celery workers |
| Security | OWASP Top 10 mitigated | TLS, JWT/RBAC, RLS, scans, secrets controls |
| Maintainability | Enforced architecture boundaries | Clean Architecture, ADRs, lint/type/architecture gates |
| Observability | Correlated logs/metrics/traces | OpenTelemetry, Prometheus, Grafana, Loki |
| Accessibility | WCAG 2.2 AA | Design system, keyboard/focus, axe-core tests |

┌──────────────────────────────────────────────────────────┐
│  3. Constraints                                           │
└──────────────────────────────────────────────────────────┘

Lansweeper API limits must be respected. Tenant isolation and auditability are mandatory.

┌──────────────────────────────────────────────────────────┐
│  4. Verification                                          │
└──────────────────────────────────────────────────────────┘

Use CI gates, k6 load tests, OWASP ZAP, architecture tests, and SLO monitoring.

┌──────────────────────────────────────────────────────────┐
│  5. Standards                                             │
└──────────────────────────────────────────────────────────┘

Use canonical NexusAsset Cloud naming (`nxa-*`, `@nexusasset/*`, `NXA_*`, `*.nexusasset.cloud`). Enforce Clean Architecture, tenant isolation (`tenant_id` + PostgreSQL RLS), and OpenAPI-first contracts.

┌──────────────────────────────────────────────────────────┐
│  6. Related Files                                         │
└──────────────────────────────────────────────────────────┘

- [../04-testing/01-test-strategy.md](../04-testing/01-test-strategy.md)
- [../06-operations/06-slo-sli.md](../06-operations/06-slo-sli.md)


┌──────────────────────────────────────────────────────────┐
└──────────────────────────────────────────────────────────┘
▸ Distributed from Fundamentals reference library (rewritten for NexusAsset Cloud)
- ═══════════════════════════════════════════════════════
- ───────────────────────────────────────────────────────
- ─────────────────────────────────────────────────────

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

╭─ NOTE ──────────────────────────────────────────────────────╮
│ Apply these practices with NexusAsset Cloud standards:      │
│ Clean Architecture, `nxa_*` contexts, OpenAPI 3.1,          │
│ `tenant_id` + PostgreSQL RLS, and security quality gates.   │
╰─────────────────────────────────────────────────────────────╯
─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: docs@nexusasset.cloud
─────────────────────────────────────────────────────────────
