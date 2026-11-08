┌──────────────────────────────────────────────────────────────┐
│  File        : 12-adr/0005-openapi-first-codegen.md         │
│  Project     : NexusAsset Cloud                              │
│  Module      : 12-adr                                        │
│  Version     : 1.0.0                                         │
│  Status      : Accepted                                      │
│  Owner       : Architecture Review Board                     │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  ADR-0005: OPENAPI-FIRST CODEGEN                               ║
║  Use OpenAPI 3.1 as the contract source of truth               ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Context](#1-context)
2. [Decision](#2-decision)
3. [Consequences](#3-consequences)
4. [Related Files](#4-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Context                                              │
└──────────────────────────────────────────────────────────┘
NexusAsset Cloud requires an enterprise-grade, tenant-safe, and maintainable platform for CMDB/ITAM, Lansweeper integration, billing, governance, web, and mobile delivery.

┌──────────────────────────────────────────────────────────┐
│  2. Decision                                             │
└──────────────────────────────────────────────────────────┘
Use OpenAPI 3.1 as the contract source of truth. Implement this decision using NexusAsset Cloud canonical naming and quality gates.

┌──────────────────────────────────────────────────────────┐
│  3. Consequences                                         │
└──────────────────────────────────────────────────────────┘
▸ Positive
- Improves consistency, maintainability, and delivery confidence.

▸ Trade-offs
- Requires discipline, tooling, and review enforcement.

┌──────────────────────────────────────────────────────────┐
│  4. Related Files                                        │
└──────────────────────────────────────────────────────────┘
- [README.md](./README.md)
- [../02-design/04-clean-architecture.md](../02-design/04-clean-architecture.md)

─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: docs@nexusasset.cloud
─────────────────────────────────────────────────────────────
