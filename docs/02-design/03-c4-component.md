# NexusAsset Cloud — C4 Components for Key Services

## Current State and Evidence
Status: Expanded baseline (Phase B/C). Evidence is Descriptor/Target/Planned unless verified by executable implementation, tests, scans, or runtime output.

## Backend Components (Target)
| Bounded context | Domain | Application | Infrastructure | Interfaces |
|---|---|---|---|---|
| `nxa_accounts` | identity/user/role concepts | auth/user use cases, ports | persistence/adapters | auth/user APIs |
| `nxa_tenants` | tenant/membership concepts | tenant context/use cases | tenant repos/RLS integration | tenant APIs/admin |
| `nxa_assets` | asset/relationship/lifecycle | CMDB use cases | repos/search/index adapters | asset APIs |
| `nxa_lansweeper` | discovery/sync/conflict concepts | sync/webhook/import use cases | Lansweeper client, mappers, job store | sync APIs/webhooks/tasks |
| `nxa_audit` | audit event concepts | record/query audit | append/store adapters | audit query APIs |
| `nxa_notifications` | notification concepts | dispatch use cases | providers/templates | APIs/tasks |
| `nxa_billing` | commercial concepts | billing use cases | billing persistence/adapters | APIs |

## Frontend Components (Target)
App routes/pages, BFF handlers, API client layer, auth/session handling, feature modules (assets, integrations, audit), shared UI/tokens.

## Status
Component inventory is target/design; verify per implemented module.
