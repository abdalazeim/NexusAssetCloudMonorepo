# NexusAsset Cloud — Product Vision, Scope, and Navigation

## Current State and Evidence
| Field | Value |
|---|---|
| Status | Expanded baseline (Phase B/C) |
| Evidence level | Descriptor/design evidence + target intent; business metrics and production usage are Unverified |
| Primary evidence | Root `README.md`, `docs/00-overview/*`, `docs/01-analysis/*`, `docs/02-design/*`, `prompts/README.md` |

## Purpose
Provide a shared, professional understanding of NexusAsset Cloud: why the platform exists, which business and technical problems it addresses, what capabilities it targets, and how readers navigate the documentation set.

## Project Identity
| Attribute | Value |
|---|---|
| Product | NexusAsset Cloud |
| Tagline | "Where Discovery Meets Enterprise" |
| Domain | Enterprise SaaS / ITAM / CMDB / governance / commercial operations |
| Current repository state | Specification and execution foundation (not a verified running production system) |
| Canonical naming | `nxa_*` contexts, `nxa-*` services, `@nexusasset/*` packages, `NXA_*` secrets, `*.nexusasset.cloud` domains |

## Problem Space (Target)
Organizations often discover technical assets through tools such as Lansweeper but struggle to convert discovery data into trusted operational, governance, and commercial decisions. NexusAsset Cloud targets this gap by unifying discovery with CMDB/ITAM workflows, tenant-safe data governance, auditability, and business context.

## Target Outcomes
| Outcome | Business value | Technical implication | Status |
|---|---|---|---|
| Unified asset visibility | Better decisions across IT, security, and finance | CMDB/asset model, integration ingestion, relationships | Target |
| Trusted tenant isolation | SaaS/compliance confidence | `tenant_id`, authorization, PostgreSQL RLS, negative tests | Target |
| Reliable Lansweeper synchronization | Accurate inventory with less manual effort | Idempotent sync, webhooks/delta/bulk, Field Ownership Matrix | Target |
| Auditable operations | Compliance and incident investigation support | Audit events, structured logs, access controls | Target |
| Governed delivery | Lower change risk | OpenAPI-first, CI gates, ADRs, runbooks | Target |

## Scope Boundaries
### In-scope (target)
- Discovery integration and normalized asset/CMDB data.
- Tenant-aware asset, identity, audit, notification, and billing contexts as defined.
- Web/BFF, API, async workers, and mobile API consumption.
- Security, observability, testing, deployment, and operations documentation/processes.

### Out-of-scope / not claimed as implemented now
- Verified production runtime, complete UI, complete OpenAPI surface, executed CI/CD, or production compliance certification.
- Any endpoint, table, job, dashboard, control, or workflow not backed by descriptor/design evidence must be treated as Target/Planned/Unverified.

## Contents
| File | Description | Expansion note |
|---|---|---|
| `01-vision.md` | Strategic vision, outcomes, principles, metrics | Expand metrics with measurable status labels. |
| `02-product-overview.md` | Capabilities, stack, differentiators | Reconcile target stack with current-state evidence. |
| `03-glossary.md` | Core terms | Add bounded-context, security, sync, and API terms. |
| `04-navigation-map.md` | Module/menu/navigation structure | Mark conceptual vs implemented navigation. |

## Reading Order
1. `README.md` (root) for repository state and entry points.
2. `00-overview/01-vision.md` and `02-product-overview.md`.
3. `01-analysis/` for requirements and domain.
4. `02-design/` and `12-adr/` for architecture decisions.
5. `03-implementation/`, `09-api/`, `07-security/`, `04-testing/`, `05-deployment/`, `06-operations/` for delivery standards.
6. `prompts/README.md` for phased execution.

## Related Files
- `../README.md`
- `../01-analysis/README.md`
- `../02-design/README.md`
- `../../prompts/README.md`
