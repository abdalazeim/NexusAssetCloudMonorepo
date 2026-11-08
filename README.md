<div align="center">

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ ╔══════════════════════════════════════════════════════════════════════════════╗
│ ║                                                                              ║
│ ║      ███╗   ██╗███████╗██╗  ██╗██╗   ██╗███████╗ █████╗ ███████╗███████╗    ║
│ ║      ████╗  ██║██╔════╝╚██╗██╔╝██║   ██║██╔════╝██╔══██╗██╔════╝██╔════╝    ║
│ ║      ██╔██╗ ██║█████╗   ╚███╔╝ ██║   ██║███████╗███████║███████╗█████╗      ║
│ ║      ██║╚██╗██║██╔══╝   ██╔██╗ ██║   ██║╚════██║██╔══██║╚════██║██╔══╝      ║
│ ║      ██║ ╚████║███████╗██╔╝ ██╗╚██████╔╝███████║██║  ██║███████║███████╗    ║
│ ║      ╚═╝  ╚═══╝╚══════╝╚═╝  ╚═╝ ╚═════╝ ╚══════╝╚═╝  ╚═╝╚══════╝╚══════╝    ║
│ ║                                                                              ║
│ ║                       N E X U S A S S E T   C L O U D                      ║
│ ║                     "Where Discovery Meets Enterprise"                       ║
│ ║                                                                              ║
│ ╚══════════════════════════════════════════════════════════════════════════════╝
└──────────────────────────────────────────────────────────────────────────────┘

| ◈ Type | ◈ Status | ◈ Version | ◈ License |
|:---:|:---:|:---:|:---:|
| Open Source — Open for Development | Specification / Phased Execution | 1.0.0 | See `project-meta/` |

</div>

---

## ◈ Project Snapshot

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ ┌──────────────────────────────────────────────────────────────────────────────┐
│ │  NexusAsset Cloud unifies Lansweeper asset discovery with Enterprise and    │
│ │  Commercial operations to deliver:                                           │
│ │                                                                              │
│ │   [CMDB]  [ITAM]  [Governance]  [Audit]  [Billing]  [Multi-Tenancy]        │
│ │   [BFF + Web]  [Mobile API]  [Observability]  [CI/CD + DevOps]             │
│ └──────────────────────────────────────────────────────────────────────────────┘
└──────────────────────────────────────────────────────────────────────────────┘

---

## ◈ Repository Geometry

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ .                                   (root: README.md only)
│ ├── README.md                       ◀ you are here
│ ├── docs/                           Enterprise documentation (00..14)
│ │   ├── 00-overview/  01-analysis/  02-design/  03-implementation/
│ │   ├── 04-testing/   05-deployment/ 06-operations/ 07-security/
│ │   ├── 08-mobile/    09-api/  10-design-system/ 11-governance/
│ │   └── 12-adr/  13-diagrams/  14-appendices/
│ ├── platform/                       Platform domains (spec/skeleton)
│ │   ├── api/  backend/  frontend/  mobile/  packages/
│ │   └── infrastructure/  tests/  scripts/
│ ├── prompts/                        Executable phased prompt pack (00..13)
│ ├── project-meta/                   Governance, policies, env/runtime descriptors
│ └── archive/                        Historical source drafts
└──────────────────────────────────────────────────────────────────────────────┘

---

## ◈ System Architecture (C4-Oriented View)

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│                          ┌─────────────────────────────────────┐
│                          │           USERS / CHANNELS          │
│                          │  [Web Admin] [Manager] [Mobile] [API]│
│                          └──────────────────┬──────────────────┘
│                                             │ HTTPS
│                                             ▼
│                          ┌─────────────────────────────────────┐
│                          │      EDGE  ·  nxa-traefik-public     │
│                          │   TLS (nxa-le) · routing · headers   │
│                          └──────────────┬───────────┬──────────┘
│                                         │           │
│                        app.nexusasset.cloud   api.nexusasset.cloud
│                                         │           │
│                                         ▼           ▼
│                          ┌──────────────────┐ ┌──────────────────────────┐
│                          │  nxa-frontend    │ │      nxa-backend         │
│                          │ Next.js 15 + BFF │ │  Django 5 + DRF (Clean)  │
│                          │ React 19 / shadcn│ │  nxa_* bounded contexts  │
│                          └────────┬─────────┘ └──────┬───────────┬───────┘
│                                   │  BFF/OpenAPI     │           │
│                                   └──────────────────►           │
│                                                      SQL/RLS       │ async
│                                                         ▼          ▼
│                                               ┌────────────┐ ┌────────────┐
│                                               │  nxa-db    │ │ nxa-redis  │
│                                               │ PostgreSQL │ │   Redis 7  │
│                                               │ 16 + RLS   │ │ cache/queue│
│                                               └────────────┘ └─────┬──────┘
│                                                                    ▼
│                                                             ┌────────────┐
│                                                             │ nxa-worker │
│                                                             │   Celery   │
│                                                             └─────┬──────┘
│                                                      GraphQL/Webhooks/Delta
│                                                                   ▼
│                                                      ┌──────────────────────┐
│                                                      │  Lansweeper Platform │
│                                                      └──────────────────────┘
└──────────────────────────────────────────────────────────────────────────────┘

---

## ◈ Clean Architecture Rule

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ ┌──────────────────────────────────────────────────────────────────────────────┐
│ │  DEPENDENCY DIRECTION  (inward only)                                         │
│ │                                                                              │
│ │   Interfaces  ───────▶  Application  ───────▶  Domain                        │
│ │   (DRF/BFF/UI)         (Use Cases/DTO/Ports)  (Entities/Rules/Events)       │
│ │                              ▲                                               │
│ │                              │ implements                                    │
│ │                        Infrastructure                                         │
│ │                  (ORM, Lansweeper, Redis, Adapters)                          │
│ │                                                                              │
│ │   ✓ Domain has no Django/ORM/framework imports                               │
│ │   ✓ OpenAPI 3.1 = API source of truth                                       │
│ │   ✓ tenant_id + PostgreSQL RLS = isolation                                   │
│ │   ✓ Lansweeper sync = idempotent + observable + Field Ownership Matrix       │
│ └──────────────────────────────────────────────────────────────────────────────┘
└──────────────────────────────────────────────────────────────────────────────┘

---

## ◈ Canonical Naming Matrix

| ◇ Area | ◇ Standard |
|---|---|
| Bounded contexts | `nxa_core`, `nxa_accounts`, `nxa_tenants`, `nxa_assets`, `nxa_lansweeper`, `nxa_billing`, `nxa_notifications`, `nxa_audit` |
| Services | `nxa-backend`, `nxa-frontend`, `nxa-worker`, `nxa-db`, `nxa-redis` |
| Edge / TLS | `nxa-traefik-public`, `nxa-le` |
| Domains | `app.nexusasset.cloud`, `api.nexusasset.cloud` |
| Packages | `@nexusasset/*` |
| Secrets / env | `NXA_*` (never committed) |
| Contract | `platform/api/contracts/openapi.yaml.description.md` (target: OpenAPI 3.1 source of truth) |

---

## ◈ Technology Stack

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ ┌──────────────┬───────────────────────────────────────────────────────────────┐
│ │ Layer        │ Target Technology                                             │
│ ├──────────────┼───────────────────────────────────────────────────────────────┤
│ │ Backend      │ Django 5 · DRF · Clean Architecture · DDD                     │
│ │ Frontend     │ Next.js 15 (App Router) · React 19 · shadcn/ui · Tailwind    │
│ │ API          │ REST · OpenAPI 3.1 · JWT · BFF                               │
│ │ Data         │ PostgreSQL 16 · Streaming Replication · RLS                   │
│ │ Async        │ Redis 7 · Celery · Celery Beat                                │
│ │ Integration  │ Lansweeper GraphQL/Webhooks/Delta/Bulk · idempotent sync      │
│ │ Edge         │ Traefik v3 · TLS · routing · rate limits · security headers   │
│ │ Mobile       │ Android (Kotlin) · iOS (Swift) · API-consuming clients        │
│ │ Delivery     │ Docker · Docker Compose · Jenkins LTS · Ansible               │
│ │ Observability│ Prometheus · Grafana · Loki · OpenTelemetry                   │
│ │ Security     │ OWASP controls · SAST/DAST/secrets/dependency scanning        │
│ └──────────────┴───────────────────────────────────────────────────────────────┘
└──────────────────────────────────────────────────────────────────────────────┘

---

## ◈ Execution Path (How to Build)

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│   START
│     │
│     ▼
│ [Read prompts/README.md] ──▶ [Apply 00-global-context-and-rules.md]
│     │
│     ▼
│ Phase 01 Foundation ─▶ 02 Backend ─▶ 03 API/OpenAPI ─▶ 04 Lansweeper
│     │
│     ▼
│ Phase 05 Frontend/BFF ─▶ 06 Mobile ─▶ 07 Testing & Quality Gates
│     │
│     ▼
│ Phase 08 DevOps (Docker/Traefik) ─▶ 09 CI/CD (Jenkins/Ansible)
│     │
│     ▼
│ Phase 10 Security ─▶ 11 Observability ─▶ 12 Docs/ADRs/Runbooks
│     │
│     ▼
│ Phase 13 Final Verification & Release  ──▶  GO / NO-GO
└──────────────────────────────────────────────────────────────────────────────┘

> Run each phase in a fresh session, verify before advancing, and do not treat CI/CD/production hardening as ready before local runtime verification passes.

---

## ◈ Quality Gates

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ ┌──────────────────────────────────────────────────────────────────────────────┐
│ │  REQUIRED GATES                                                              │
│ │  [✓] Lint/Format      [✓] Type Checks      [✓] Architecture (Dependency Rule) │
│ │  [✓] Unit Tests       [✓] Integration      [✓] Contract/OpenAPI Alignment     │
│ │  [✓] Security Scans   [✓] Build/Package    [✓] Codegen Consistency           │
│ └──────────────────────────────────────────────────────────────────────────────┘
└──────────────────────────────────────────────────────────────────────────────┘

---

## ◈ Documentation Entry Points

| ◇ Need | ◇ Path |
|---|---|
| Project overview | `docs/00-overview/README.md` |
| Domain & requirements | `docs/01-analysis/README.md` |
| Architecture & design | `docs/02-design/README.md` |
| Implementation standards | `docs/03-implementation/README.md` |
| API guidance | `docs/09-api/README.md` |
| Security | `docs/07-security/README.md` |
| Operations | `docs/06-operations/README.md` |
| ADRs | `docs/12-adr/README.md` |
| Diagrams | `docs/13-diagrams/README.md` |
| Execution prompts | `prompts/README.md` |
| Governance/policies | `project-meta/` |

---

## ◈ Open Source Notice

This repository is presented as an **open-source, development-ready specification and execution foundation** for NexusAsset Cloud. Contributors should follow the governance and security materials in `project-meta/`, preserve canonical naming, respect tenant isolation/security rules, and use the phased prompts to implement production artifacts safely.

---

<div align="center">

┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ ┌──────────────────────────────────────────────────────────────────────────────┐
│ │                                                                              │
│ │                     This Script By: ☠αвɒ єℓαzєм☠                             │
│ │                              Thanks For Usage :)                              │
│ │                           Have A Nice Day,GoodBye :)                          │
│ │                                                                              │
│ └──────────────────────────────────────────────────────────────────────────────┘
└──────────────────────────────────────────────────────────────────────────────┘

</div>
