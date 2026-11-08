┌──────────────────────────────────────────────────────────────┐
│  File        : project-meta/CONTRIBUTING.md                 │
│  Project     : NexusAsset Cloud                              │
│  Version     : 1.0.0                                         │
│  Status      : Stable                                        │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

# Contributing to NexusAsset Cloud

## Purpose
This guide defines how to propose changes safely and consistently to NexusAsset Cloud.

## Branching and Commits
- Use short-lived branches: `feat/`, `fix/`, `chore/`, `docs/`, `refactor/`, `hotfix/`.
- Use Conventional Commits.
- Keep pull requests focused, reviewable, and linked to an issue/task when applicable.

## Required Quality Gates
- Linting and formatting checks
- Type checks
- Architecture checks (Clean Architecture dependency rule)
- Unit/integration tests (and contract/E2E as applicable)
- Security scans (secrets, dependency/SAST; DAST on staging where configured)
- Build/package verification
- OpenAPI/codegen consistency when API contracts change

## Security and Secrets
- Never commit secrets or credentials.
- Use approved secret stores and `NXA_*` environment variable references only.
- Report security issues through `project-meta/SECURITY.md`; do not open public issues for sensitive findings.

## Documentation
- Update `README.md`, `docs/`, ADRs, runbooks, and `.description.md` specifications when behavior, structure, or contracts change.
- Preserve canonical NexusAsset Cloud naming: `nxa_*`, `nxa-*`, `@nexusasset/*`, `NXA_*`, `*.nexusasset.cloud`.

─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
─────────────────────────────────────────────────────────────
