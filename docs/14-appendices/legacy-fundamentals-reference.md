┌──────────────────────────────────────────────────────────────┐
│  File        : docs/14-appendices/legacy-fundamentals-reference.md │
│  Project     : NexusAsset Cloud                              │
│  Version     : 1.0.0                                         │
│  Status      : Archived (English Summary)                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

# Legacy Fundamentals Reference (English Summary)

## Purpose
This appendix preserves an English-only summary of legacy software-development fundamentals that informed NexusAsset Cloud documentation. The original mixed-language draft is not retained as product documentation.

## Covered Topics (Summary)
- Planning and requirements analysis
- System architecture and design patterns
- Clean Architecture and modularization
- API design and integration
- Data modeling, transactions, and consistency
- Quality, testing, security, performance, and operations
- Documentation and governance practices

## NexusAsset Cloud Application
All reused guidance must be applied through NexusAsset Cloud standards:
- Canonical naming (`nxa_*`, `nxa-*`, `@nexusasset/*`, `NXA_*`, `*.nexusasset.cloud`)
- Clean Architecture and DDD
- OpenAPI 3.1 contract-first APIs
- Multi-tenancy with `tenant_id` and PostgreSQL RLS
- Secure secret management and idempotent Lansweeper synchronization

─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
─────────────────────────────────────────────────────────────
