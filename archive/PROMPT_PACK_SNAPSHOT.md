# NexusAsset Cloud — Prompt Pack Snapshot (Phases 00–13)

## Purpose
Preserved snapshot of the phased execution intent. Authoritative, editable prompt files live in `prompts/`.

## Phase Map
| Phase | Focus | Intent |
|---:|---|---|
| 00 | Global context and rules | Establish non-negotiable standards and safety constraints. |
| 01 | Foundation and tooling | Create baseline project/tooling foundation. |
| 02 | Backend Clean Architecture | Implement `nxa_*` bounded contexts with dependency rule enforcement. |
| 03 | API/OpenAPI and contracts | Make OpenAPI 3.1 the contract source of truth. |
| 04 | Lansweeper integration | Deliver idempotent sync, webhooks/delta/bulk handling, Field Ownership Matrix. |
| 05 | Frontend and BFF | Build Next.js/React web experience with secure BFF boundaries. |
| 06 | Mobile Android | Build Kotlin Android baseline aligned to contracts. |
| 07 | Testing and quality gates | Establish unit/integration/contract/architecture/security checks. |
| 08 | DevOps Docker/Traefik | Provide local runtime with canonical services and edge routing. |
| 09 | CI/CD Jenkins/Ansible | Automate quality-gated delivery. |
| 10 | Security and compliance | Enforce RBAC, secrets, tenant isolation, scanning, and hardening. |
| 11 | Observability and operations | Add logs/metrics/traces, alerts, and runbooks. |
| 12 | Documentation, ADRs, runbooks | Align docs with implemented reality. |
| 13 | Final verification and release | Produce evidence-based Go/No-Go readiness. |

## Execution Rules Snapshot
- Use fresh sessions per phase where practical.
- Do not skip verification gates.
- Do not claim CI/CD or production readiness before local runtime and security gates pass.
- Keep OpenAPI, naming, tenant isolation, and auditability consistent.
