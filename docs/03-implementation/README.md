┌──────────────────────────────────────────────────────────────┐
│  File        : 03-implementation/README.md                   │
│  Project     : NexusAsset Cloud                              │
│  Module      : 03-implementation                             │
│  Version     : 1.0.0                                         │
│  Status      : Stable                                        │
│  Owner       : Platform Architecture                         │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  IMPLEMENTATION                                                ║
║  Project Structure, Standards, and Engineering Patterns        ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Purpose](#1-purpose)
2. [Contents](#2-contents)
3. [Implementation Rules](#3-implementation-rules)
4. [Related Files](#4-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Purpose                                              │
└──────────────────────────────────────────────────────────┘
This module defines how NexusAsset Cloud code is organized and implemented. It translates architecture decisions into enforceable repository structure, coding conventions, and implementation patterns.

┌──────────────────────────────────────────────────────────┐
│  2. Contents                                             │
└──────────────────────────────────────────────────────────┘
| File | Description |
|---|---|
| [01-project-structure.md](./01-project-structure.md) | Canonical monorepo and backend app layout |
| [02-naming-conventions.md](./02-naming-conventions.md) | Naming standards across code, APIs, infra, and docs |
| [03-solid-principles.md](./03-solid-principles.md) | SOLID (Single responsibility, Open/closed, Liskov substitution, Interface segregation, Dependency inversion) application |
| [04-clean-code.md](./04-clean-code.md) | Readability, maintainability, and code hygiene |
| [05-dependency-rule.md](./05-dependency-rule.md) | Dependency direction and enforcement |
| [06-use-case-pattern.md](./06-use-case-pattern.md) | Application use-case implementation pattern |
| [07-repository-pattern.md](./07-repository-pattern.md) | Ports, repositories, and persistence boundaries |
| [08-mappers.md](./08-mappers.md) | Domain/DTO/ORM mapping rules |
| [09-migrations-strategy.md](./09-migrations-strategy.md) | Expand/Contract database migrations |
| [10-error-handling.md](./10-error-handling.md) | Domain/application/API error mapping |
| [11-transactions.md](./11-transactions.md) | Unit of Work and transactional boundaries |
| [12-feature-flags.md](./12-feature-flags.md) | Feature flag lifecycle and governance |

┌──────────────────────────────────────────────────────────┐
│  3. Implementation Rules                                 │
└──────────────────────────────────────────────────────────┘
- Keep Domain independent from frameworks.
- Use `nxa_*` bounded-context apps under `backend/apps/`.
- Use `nxa-*` service/container names in infrastructure.
- Treat `api/contracts/openapi.yaml` as the single source of truth.
- Enforce tenant isolation and architecture rules in CI.

┌──────────────────────────────────────────────────────────┐
│  4. Related Files                                        │
└──────────────────────────────────────────────────────────┘
- [../02-design/04-clean-architecture.md](../02-design/04-clean-architecture.md)
- [../04-testing/09-architecture-testing.md](../04-testing/09-architecture-testing.md)
- [../11-governance/01-git-workflow.md](../11-governance/01-git-workflow.md)

─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: docs@nexusasset.cloud
─────────────────────────────────────────────────────────────
