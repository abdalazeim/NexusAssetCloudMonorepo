# NexusAsset Cloud — Secrets Management and Rotation

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline |
| Evidence level | Implemented (Descriptor Evidence) for `NXA_*` naming/template intent; runtime secret-store integration Unverified |
| Primary evidence | `project-meta/.env.example.description.md`, `project-meta/.gitignore.description.md`, `project-meta/SECURITY.md`, `prompts/10-security-and-compliance.md` |

## Purpose
Define how NexusAsset Cloud names, stores, accesses, rotates, audits, and protects secrets and sensitive configuration values.

## Secret Classes
| Class | Examples | Risk |
|---|---|---|
| Application secrets | `NXA_SECRET_KEY`, JWT/signing keys | Critical |
| Data credentials | PostgreSQL/Redis credentials/URLs | Critical |
| Integration secrets | `NXA_LANSWEEPER_TOKEN`, webhook secrets | Critical |
| Delivery secrets | CI credentials, registry tokens, Ansible/Vault secrets | Critical |
| Observability/access | API keys/tokens for telemetry or admin systems | High |

## Mandatory Rules
1. Use `NXA_*` naming for NexusAsset Cloud environment/secret keys.
2. Never commit real secrets, private keys, tokens, or production credentials.
3. Never print secrets in logs, stack traces, CI output, or client responses.
4. Use least privilege and scope credentials to the minimum required tenant/system/action.
5. Rotate secrets on compromise suspicion, personnel/process events (as policy requires), and scheduled intervals for critical credentials.
6. Separate local development templates (`.env.example` descriptor intent) from real secret stores.

## Lifecycle
| Stage | Control (Target) | Status |
|---|---|---|
| Create | Generate strong secret in approved manager; record owner/purpose | Unverified |
| Store | Secret store/Vault/CI credential store; encrypted at rest | Unverified |
| Access | Runtime injection; least-privilege identities/services | Unverified |
| Use | Masked logs; scoped usage; timeout/retry safety | Unverified |
| Rotate | Planned rotation + emergency rotation runbook | Unverified |
| Revoke | Immediate invalidation on leak/compromise | Unverified |
| Audit | Access/change events where supported | Unverified |

## Rotation Priorities (Target)
| Secret | Trigger examples | Special care |
|---|---|---|
| Lansweeper token/webhook secret | Leak, staff/process change, suspected abuse | Coordinate sync/webhook verification to avoid outage; verify idempotent retries |
| JWT/signing keys | Policy schedule, compromise | Support safe rollover if multiple keys/alg strategy implemented |
| DB credentials | Compromise, rotation schedule | Coordinate migrations/workers; avoid downtime |
| CI/deploy credentials | Leak, runner compromise, scope change | Revoke quickly; review pipeline history |

## Evidence Requirements
- Secret scanning configuration/results.
- `.gitignore`/repository hygiene controls.
- Secret-store integration docs/config references (when implemented).
- Rotation runbook and incident response linkage.

## Related Files
- `01-threat-model.md`
- `../06-operations/08-incident-response.md`
- `../../project-meta/.env.example.description.md`
- `../../project-meta/.gitignore.description.md`
