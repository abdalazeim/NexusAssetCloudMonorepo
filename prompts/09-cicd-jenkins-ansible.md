# Phase 09 — CI/CD (Jenkins + Ansible)

## Objective
Implement NexusFlow CI/CD automation using Jenkins LTS and Ansible for NexusAsset Cloud, with quality gates, artifact/image handling, and controlled deployments.

## Preconditions
- Phases 07–08 pass locally. Do not create production deployment behavior before local verification is reliable.

## Required Context
- `infrastructure/jenkins/` (Jenkinsfile, shared libs/config if present)
- `infrastructure/ansible/`
- `Makefile`/scripts/quality gate commands
- `05-deployment/04-jenkins.md` and related docs

## Deliverables
1. **Jenkins pipelines**
   - Declarative Jenkinsfiles (multibranch-friendly) for backend, frontend, and/or monorepo orchestration.
   - Stages: checkout, setup, lint, typecheck, tests, architecture/contract checks, security scans, build images/packages, publish artifacts/images (environment-gated), deploy staging (gated), manual/approved prod promotion.
2. **Quality and security gates**
   - Fail fast on lint/type/test/contract/architecture failures.
   - Integrate Trivy/Bandit/Safety/Gitleaks/SonarQube/OWASP ZAP hooks where appropriate; distinguish PR, main, staging, and nightly schedules.
3. **Ansible deployment**
   - Inventories/playbooks/roles for staging/prod-shaped deployment using Docker Compose or the approved runtime target.
   - Idempotent tasks, secrets via Jenkins credentials/Ansible Vault/external secret store references (never plaintext in repo), health checks, and rollback notes/hooks.
4. **Release controls**
   - Version/tag strategy, image naming, environment promotion rules, approvals for production, and notifications.
5. **Documentation**
   - Update Jenkins/Ansible runbooks with prerequisites, credentials map (`NXA_*`/CI secrets), and recovery steps.

## Verification
┌─ Specification Block (converted from `text`; scope 2 — no fenced code blocks) ─┐
│ - Jenkinsfile syntax/lint check (available Jenkins tooling or declarative validator)
│ - ansible-playbook --syntax-check ... 
│ - ansible-lint ... (if available)
│ - Dry-run or staging-safe verification steps documented
└──────────────────────────────────────────────────────────────────────────────┘
If Jenkins/Ansible runtimes are unavailable locally, perform static validation and provide exact execution commands.

## Exit Criteria
- CI/CD definitions cover required NexusAsset Cloud gates.
- Deployment automation is idempotent and secret-safe.
- Promotion to production requires explicit controls.
- Runbooks explain operation and rollback.
