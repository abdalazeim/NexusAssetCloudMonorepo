┌──────────────────────────────────────────────────────────────┐
│  File        : 12-adr/README.md                             │
│  Project     : NexusAsset Cloud                              │
│  Module      : 12-adr                                       │
│  Version     : 1.0.0                                        │
│  Status      : Stable                                       │
│  Owner       : Platform Engineering                          │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  ARCHITECTURE DECISION RECORDS                                 ║
║  Key Architectural Decisions                                   ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Purpose](#1-purpose)
2. [Contents](#2-contents)
3. [Related Files](#3-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Purpose                                              │
└──────────────────────────────────────────────────────────┘
This module contains NexusAsset Cloud documentation for **key architectural decisions**.

┌──────────────────────────────────────────────────────────┐
│  2. Contents                                             │
└──────────────────────────────────────────────────────────┘
| File | Description |
|---|---|
| [0001-use-clean-architecture.md](0001-use-clean-architecture.md) | Use Clean Architecture and DDD boundaries |
| [0002-adopt-nextjs-app-router.md](0002-adopt-nextjs-app-router.md) | Adopt Next.js 15 App Router |
| [0003-use-traefik-instead-of-nginx.md](0003-use-traefik-instead-of-nginx.md) | Use Traefik v3 as edge proxy |
| [0004-bff-pattern.md](0004-bff-pattern.md) | Use BFF for web authentication and aggregation |
| [0005-openapi-first-codegen.md](0005-openapi-first-codegen.md) | Use OpenAPI-first contract and code generation |
| [0006-shadcn-ui.md](0006-shadcn-ui.md) | Use shadcn/ui and Tailwind for web UI primitives |
| [0007-lansweeper-hybrid-sync.md](0007-lansweeper-hybrid-sync.md) | Use hybrid Lansweeper sync with field ownership |
| [0008-multi-tenancy-strategy.md](0008-multi-tenancy-strategy.md) | Use shared schema with tenant_id and PostgreSQL RLS |
| [0009-event-driven-architecture.md](0009-event-driven-architecture.md) | Use lightweight domain events and async workers |
| [0010-jenkins-over-gitlab-ci.md](0010-jenkins-over-gitlab-ci.md) | Use Jenkins LTS for CI/CD orchestration |

┌──────────────────────────────────────────────────────────┐
│  3. Related Files                                        │
└──────────────────────────────────────────────────────────┘
- [../README.md](../README.md)
- [../00-overview/README.md](../00-overview/README.md)
- [../03-implementation/README.md](../03-implementation/README.md)

─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: docs@nexusasset.cloud
─────────────────────────────────────────────────────────────
