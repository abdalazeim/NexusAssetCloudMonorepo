┌──────────────────────────────────────────────────────────────┐
│  File        : 14-appendices/03-references.md                │
│  Project     : NexusAsset Cloud                              │
│  Module      : 14-appendices                                 │
│  Version     : 1.1.0                                         │
│  Status      : Stable                                        │
│  Owner       : Documentation & Architecture                  │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  REFERENCES                                                    ║
║  Source Materials and Distributed Fundamentals                ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Purpose](#1-purpose)
2. [Archived NexusAsset Draft Sources](#2-archived-nexusasset-draft-sources)
3. [Fundamentals Library Distribution](#3-fundamentals-library-distribution)
4. [External/Binary References](#4-externalbinary-references)
5. [Usage Rules](#5-usage-rules)
6. [Related Files](#6-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Purpose                                              │
└──────────────────────────────────────────────────────────┘
This file lists archived NexusAsset draft sources and records how the general `FundamentalsOfDevelopingAnySoftwareSystem` reference library was rewritten, distributed across NexusAsset Cloud documentation modules, and removed as a standalone folder.

┌──────────────────────────────────────────────────────────┐
│  2. Archived NexusAsset Draft Sources                    │
└──────────────────────────────────────────────────────────┘
| File | Source Type | Notes |
|---|---|---|
| [archived-source-01-requirements-analysis.txt](./archived-source-01-requirements-analysis.txt) | Draft | Requirements analysis (non-functional) |
| [archived-source-02-system-architecture.txt](./archived-source-02-system-architecture.txt) | Draft | System architecture (C4 context) |
| [archived-source-03-design-patterns.txt](./archived-source-03-design-patterns.txt) | Draft | Design patterns / repository guidance |
| [archived-source-04-performance-risk.txt](./archived-source-04-performance-risk.txt) | Draft | Performance analysis / risk register |
| [archived-source-05-code-documentation.txt](./archived-source-05-code-documentation.txt) | Draft | Code documentation / product overview |
| [legacy-fundamentals-reference.md](./legacy-fundamentals-reference.md) | Archived | Previous root README (non-NexusAsset identity) |

┌──────────────────────────────────────────────────────────┐
│  3. Fundamentals Library Distribution                    │
└──────────────────────────────────────────────────────────┘
The `FundamentalsOfDevelopingAnySoftwareSystem` library (64 files) was reviewed and its guidance was distributed into NexusAsset Cloud modules according to topic (analysis, design, implementation, testing, deployment, operations, security, API, design system, governance, diagrams, appendices).

| Artifact | Purpose |
|---|---|
| [fundamentals-distribution-manifest.md](./fundamentals-distribution-manifest.md) | Source-to-target distribution manifest |
| NexusAsset module files (`00`–`13`) | Canonical English standards with adapted `Adapted Source` sections |
| Standalone folder | Deleted after distribution (per request) |

╭─ NOTE ──────────────────────────────────────────────────────╮
│ NexusAsset Cloud binding standards take precedence:         │
│ Clean Architecture, `nxa_*` bounded contexts, OpenAPI 3.1, │
│ `tenant_id` + PostgreSQL RLS, `NXA_*` secrets, and CI/CD   │
│ security/quality gates.                                     │
╰─────────────────────────────────────────────────────────────╯

┌──────────────────────────────────────────────────────────┐
│  4. External/Binary References                           │
└──────────────────────────────────────────────────────────┘
| File | Type | Notes |
|---|---|---|
| `Software Engineering/Software Engineering.pdf` | PDF (workspace root reference) | Binary reference; not rewritten. Cite when used; do not treat as NexusAsset contract. |

┌──────────────────────────────────────────────────────────┐
│  5. Usage Rules                                          │
└──────────────────────────────────────────────────────────┘
- Cite sources when extracting content.
- Normalize extracted content to NexusAsset Cloud naming, English documentation standards, and architecture/security rules.
- Do not reintroduce legacy names: `PROJECT/`, `config/`, `apps/accounts/`, `apps/core/`, `web` service name, `traefik-public`, `SECRET_KEY`, or `@project/*`.

┌──────────────────────────────────────────────────────────┐
│  6. Related Files                                        │
└──────────────────────────────────────────────────────────┘
- [README.md](./README.md)
- [fundamentals-distribution-manifest.md](./fundamentals-distribution-manifest.md)
- [../README.md](../README.md)
- [../CONVENTIONS.md](../CONVENTIONS.md)

─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: docs@nexusasset.cloud
─────────────────────────────────────────────────────────────
