# NexusAsset Cloud — Jenkins Pipeline Description

**Original artifact:** `Jenkinsfile` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Defines the NexusAsset Cloud multibranch CI/CD pipeline skeleton.

## Inputs / Parameters
Repository, Jenkins credentials, branch/PR context, tools, registry, deployment targets.

## Outputs / Behavior
Pipeline results, quality reports, artifacts/images, gated deployments.

## Detailed Description
Declarative pipeline stages: Checkout (source control), Quality (lint, type checks, architecture checks, tests, security scans), Build (build `nxa-backend`, `nxa-frontend`, `nxa-worker` artifacts/images), Deploy (deploy via approved automation with controls). Full implementation adds credentials, conditions, gates, publishing, approvals, and notifications.

## NexusAsset Cloud Constraints
- No production deploy before local/runtime and security gates pass.
- Use secret stores/credentials only.
- Critical gate failures must stop the pipeline.
