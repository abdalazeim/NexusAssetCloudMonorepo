┌──────────────────────────────────────────────────────────────┐
│  File        : 05-deployment/04-jenkins.md                  │
│  Project     : NexusAsset Cloud                              │
│  Module      : 05-deployment                                │
│  Version     : 1.0.0                                         │
│  Status      : Draft                                         │
│  Owner       : DevOps/SRE                                   │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  04 JENKINS                                                    ║
║  Deployment and delivery runbooks                              ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Purpose](#1-purpose)
2. [Standard](#2-standard)
3. [Controls and Checklist](#3-controls-and-checklist)
4. [Related Files](#4-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Purpose                                              │
└──────────────────────────────────────────────────────────┘
Define the NexusAsset Cloud standard for **04 jenkins**.

┌──────────────────────────────────────────────────────────┐
│  2. Standard                                             │
└──────────────────────────────────────────────────────────┘
Apply NexusAsset Cloud canonical naming and boundaries:
- `nexusasset-cloud/`, `backend/nexusasset/`, `backend/apps/nxa_*`
- `nxa-backend`, `nxa-frontend`, `nxa-worker`, `nxa-db`, `nxa-redis`
- `app.nexusasset.cloud`, `api.nexusasset.cloud`, `nxa-traefik-public`, `nxa-le`
- `@nexusasset/*` packages and `NXA_*` secrets
- OpenAPI 3.1 source of truth, tenant isolation (`tenant_id` + PostgreSQL RLS), Clean Architecture, idempotent integrations

┌──────────────────────────────────────────────────────────┐
│  3. Controls and Checklist                               │
└──────────────────────────────────────────────────────────┘
| Control | Status | Evidence |
|---|---|---|
| Canonical naming applied | ○ In Progress | PR/docs review |
| Security and tenant isolation addressed | ○ In Progress | Tests/policies |
| CI/CD or operational gate defined | ○ In Progress | Pipeline/runbook |
| Related ADRs/docs linked | ○ In Progress | Document links |

┌──────────────────────────────────────────────────────────┐
│  4. Related Files                                        │
└──────────────────────────────────────────────────────────┘
- [README.md](./README.md)
- [../00-overview/README.md](../00-overview/README.md)
- [../03-implementation/01-project-structure.md](../03-implementation/01-project-structure.md)
- [../12-adr/README.md](../12-adr/README.md)

─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: docs@nexusasset.cloud
─────────────────────────────────────────────────────────────
