# NexusAsset Cloud — Documentation Status Taxonomy

## Purpose
This taxonomy prevents speculation. Every significant claim in NexusAsset Cloud documentation must be labeled explicitly when its evidence level is not obvious.

## Status Labels

| Status | Meaning | Allowed use |
|---|---|---|
| Implemented (Verified) | Confirmed by executable repository evidence and, where possible, observed behavior/output. | State as implemented fact with evidence reference. |
| Implemented (Descriptor Evidence) | Confirmed by approved `*.description.md` or specification artifact in the current repository state. | State as specified/descriptor-backed; do not claim runtime proof. |
| Target | Required or intended design for NexusAsset Cloud; not yet implemented. | State as target architecture/behavior. |
| Planned | Accepted future scope with known purpose; sequencing/dependency may exist. | State as planned work, not current capability. |
| Unverified | Cannot be confirmed from available evidence. | Must not be presented as fact; document what evidence is missing. |

## Application Rules
1. Current repository state is primarily **specification/execution foundation**; most runtime behaviors are Target, Planned, or Descriptor Evidence unless proven.
2. Do not invent endpoints, tables, environment variables, commands, policies, or controls.
3. Preserve exact identifiers when they exist (`nxa_*`, `NXA_*`, `@nexusasset/*`, `*.nexusasset.cloud`).
4. When expanding docs, include a "Current State and Evidence" section or status markers.
5. Security and tenant-isolation claims require the strongest available evidence label.

## Evidence Sources (Current Repository)
- Root: `README.md`, evaluation/summary Markdown files.
- `docs/`: structured module documentation and ADRs.
- `prompts/`: phased implementation intent and constraints.
- `project-meta/`: governance/policy and converted environment/runtime descriptors.
- `platform/`: converted target implementation descriptors (`*.description.md`).
- `archive/`: historical traceability only; non-authoritative.
