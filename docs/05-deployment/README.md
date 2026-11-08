# NexusAsset Cloud — Build, Release, and Environment Delivery

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Baseline documentation expanded (Phase B/C in progress) |
| Evidence level | Descriptor Evidence + Target; no verified deployment execution in current repository state |
| Primary evidence | `project-meta/docker-compose.yml.description.md`, `platform/backend/Dockerfile.description.md`, `platform/frontend/Dockerfile.description.md`, `platform/infrastructure/jenkins/Jenkinsfile.description.md`, `project-meta/Makefile.description.md`, `prompts/08-devops-docker-traefik.md`, `prompts/09-cicd-jenkins-ansible.md` |
| Constraint | Standalone code/config artifacts were converted to `*.description.md` by approved policy. Do not claim a runnable stack until executable artifacts are regenerated and verified. |

## Purpose
Define NexusAsset Cloud environment, build, packaging, deployment, and release standards for local, staging, and production delivery.

## Target Runtime Topology
| Component | Canonical name | Responsibility | Status |
|---|---|---|---|
| Web/BFF | `nxa-frontend` | User interface and BFF boundary | Target/Descriptor |
| API | `nxa-backend` | Django/DRF API and business application services | Target/Descriptor |
| Worker | `nxa-worker` | Celery async jobs (sync, notifications, etc.) | Target/Descriptor |
| Database | `nxa-db` | PostgreSQL 16 data store | Target/Descriptor |
| Cache/queue broker | `nxa-redis` | Redis 7 cache/broker | Target/Descriptor |
| Edge proxy | `nxa-traefik-public` + `nxa-le` | TLS, routing, edge controls | Target/Descriptor |

## Deployment Principles
1. Immutable build artifacts/images per release candidate.
2. Environment-specific configuration via `NXA_*` variables and secret stores; never commit secrets.
3. Database migrations are controlled, reviewed, and reversible where feasible.
4. Health checks and smoke tests gate promotion.
5. Rollback procedure is defined before production release.
6. CI/CD and production hardening do not bypass local/runtime and security verification.

## Environment Model (Target)
| Environment | Purpose | Controls |
|---|---|---|
| Local/dev | Developer implementation and fast feedback | Non-production data, local secrets/templates, mocked external dependencies where useful |
| CI | Automated build/test/security gates | Ephemeral resources, no production secrets |
| Staging | Pre-production validation, UAT, DAST/perf baseline | Sanitized data, production-like config, controlled access |
| Production | Live tenant workloads | Strict change control, approvals, monitoring, backups, incident response |

## Release Checklist (Target)
- Scope/changes approved and documented.
- OpenAPI/contract changes reviewed.
- Migrations reviewed for safety and rollback impact.
- Tests and security gates passed (with evidence).
- Images/artifacts built and tagged.
- Staging smoke/health checks passed.
- Backup/rollback readiness confirmed.
- Observability dashboards/alerts verified.
- Release notes/changelog updated.

## Contents
| File | Description | Audit note |
|---|---|---|
| `01-environments.md` | Environment definitions | Expand with config matrix and promotion flow. |
| `02-docker.md` | Docker/Compose standards | Reconcile with descriptor-based state and regeneration path. |
| `03-traefik.md` | Traefik routing/TLS/middleware | Expand with domain/TLS/header/rate-limit design. |
| `04-jenkins.md` | Jenkins pipelines/gates | Expand with stages, credentials, approvals, artifact flow. |
| `05-ansible.md` | Ansible provisioning/deployment | Expand with inventory/roles/secrets/rollback. |
| `06-ubuntu-server.md` | Ubuntu baseline | Expand hardening/patching/runtime prerequisites. |
| `07-postgresql.md` | PostgreSQL deployment/ops | Expand backups, replication, RLS migration considerations. |
| `08-redis-celery.md` | Redis/Celery deployment | Expand queues, workers, retries, monitoring. |
| `09-release-process.md` | Release/versioning process | Expand decision points and evidence checklist. |
| `10-rollback.md` | Rollback procedures | Expand app/DB/config rollback and communication steps. |

## Related Files
- `../06-operations/README.md`
- `../07-security/README.md`
- `../../prompts/08-devops-docker-traefik.md`
- `../../prompts/09-cicd-jenkins-ansible.md`
- `../../project-meta/docker-compose.yml.description.md`
- `../../project-meta/Makefile.description.md`
