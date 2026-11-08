# NexusAsset Cloud — Archive Inventory

## Status
This inventory reflects the archive as a traceability area. The repository underwent restructuring and normalization; some original draft files were summarized or replaced by authoritative documents in `docs/`, `prompts/`, and root evaluation files.

## Archive Files
| File | Type | Status | Notes |
|---|---|---|---|
| `README.md` | Index/policy | Active | Archive purpose, authority boundaries, and rules. |
| `ARCHIVE_INVENTORY.md` | Inventory | Active | Tracks archive contents and authoritative mappings. |
| `REPOSITORY_TRANSFORMATION_LOG.md` | Transformation log | Active | High-level record of repository cleanup/organization actions. |
| `HISTORICAL_SOURCE_NOTES.md` | Provenance notes | Active | Explains historical draft handling and normalization. |
| `CANONICAL_NAMING_SNAPSHOT.md` | Standards snapshot | Active | Preserved naming baseline for traceability. |
| `PROMPT_PACK_SNAPSHOT.md` | Execution snapshot | Active | Preserved phase intent for prompts 00–13. |
| `EVALUATION_CONTEXT_SNAPSHOT.md` | Evaluation snapshot | Active | Context for root evaluation Markdown files. |

## Related Authoritative Locations
| Historical/Archived Concern | Authoritative Location |
|---|---|
| Product vision and overview | `docs/00-overview/` |
| Requirements, domain, bounded contexts | `docs/01-analysis/` |
| Architecture, C4, tenancy, integration | `docs/02-design/` |
| Implementation structure and patterns | `docs/03-implementation/` |
| API contract intent | `platform/api/contracts/openapi.yaml.description.md`, `docs/09-api/` |
| Security, operations, deployment | `docs/07-security/`, `docs/06-operations/`, `docs/05-deployment/` |
| ADRs | `docs/12-adr/` |
| Phased execution | `prompts/README.md`, `prompts/00-13` |
| Governance/policy descriptors | `project-meta/` |
| Comprehensive evaluation | `NEXUSASSET_CLOUD_COMPREHENSIVE_EVALUATION.md` (root) |
| Current-state evaluation (no execution) | `NEXUSASSET_CLOUD_CURRENT_STATE_EVALUATION_NO_EXECUTION.md` (root) |
| Arabic business/technical summary | `NEXUSASSET_CLOUD_ARABIC_BUSINESS_TECHNICAL_SUMMARY.md` (root) |

## Gaps / Follow-up
- If original raw draft files are recovered from external backups, store them here as clearly labeled historical sources with English provenance notes and no secrets.
- Do not restore raw mixed-language drafts into `docs/` as active specifications without normalization.
