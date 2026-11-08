# NexusAsset Cloud — Code Ownership Description

**Original artifact:** `CODEOWNERS` (converted to English description; original file removed per approved scope 1).

## Purpose
Defines required reviewers/owners for repository paths.

## Inputs / Parameters
Pull requests touching owned paths.

## Outputs / Behavior
Reviewer assignment and review requirements in supported Git platforms.

## Detailed Description
Ownership rules map critical areas to responsible teams/users: governance/security docs, backend bounded contexts (`nxa_*`), OpenAPI contract, frontend/BFF, mobile, infrastructure/Traefik/Docker/Ansible/Jenkins, observability, and tests. Sensitive paths require specialized review.

## NexusAsset Cloud Constraints
- Keep owners current.
- API contract and security/infrastructure changes require specialized review.
