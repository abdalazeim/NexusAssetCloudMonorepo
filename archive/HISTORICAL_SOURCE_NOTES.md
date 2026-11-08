# NexusAsset Cloud — Historical Source Notes

## Purpose
This document explains how historical drafts and mixed-language inputs were handled during NexusAsset Cloud normalization.

## Handling Principles
1. Preserve intent and traceability where useful.
2. Do not keep mixed-language or ambiguous draft text as authoritative product documentation.
3. Convert relevant guidance into NexusAsset Cloud canonical naming and architecture standards.
4. Keep secrets out of all archived or active files.

## Observed Historical Input Patterns
| Pattern | Handling Approach | Authoritative Outcome |
|---|---|---|
| Raw prompt/draft files (including legacy draft naming patterns) | Consolidate, translate/normalize intent, and remove obsolete duplicates where appropriate. | `prompts/00-13`, `docs/`, and root evaluation/summary files. |
| Mixed Arabic/English planning text | Convert to English for maintained docs/specs; provide dedicated Arabic summary as a root communication artifact when explicitly requested. | English maintained docs + `NEXUSASSET_CLOUD_ARABIC_BUSINESS_TECHNICAL_SUMMARY.md`. |
| Fundamentals/reference guidance reused in docs | Normalize to NexusAsset Cloud context and remove Arabic source fragments from maintained docs. | `docs/` modules and `docs/14-appendices/` English summaries/manifests. |
| Executable/config skeletons during no-code/spec phase | Convert to `*.description.md` where Scope 1 applied. | `platform/` and `project-meta/` descriptive artifacts. |

## Traceability Guidance
When restoring or importing historical sources:
- Label the source, date/context (if known), and confidence level.
- Record where its content was mapped.
- Mark sensitive or uncertain content for review.
- Never place raw historical drafts into `docs/` as active specifications without normalization.
