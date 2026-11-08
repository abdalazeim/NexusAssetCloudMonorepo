# NexusAsset Cloud — Pagination, Filtering, and Sorting

## Status
Target/Unverified.

## Requirements (Target)
| Concern | Requirement |
|---|---|
| Pagination | Consistent envelope (items + page/cursor metadata) for list endpoints |
| Limits | Maximum page size enforced; defaults documented |
| Ordering | Deterministic default sort; sortable fields explicitly allowed |
| Filtering | Allowed filters validated; tenant scope always applied server-side |
| Performance | Index/query design supports common filters/sorts; avoid unbounded scans |

## Evidence Needed
OpenAPI schemas/parameters, implementation, and integration/performance tests.
