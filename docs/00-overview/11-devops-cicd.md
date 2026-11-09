╔═════════════════════════════════════════════════════════════════════════════════════════════════╗
║ Document ID: DOC-11                                                                            ║
║ Title: DevOps & CI/CD — Jenkins, Ansible, GitOps, Promotion, Rollback                          ║
║ Version: 1.0.0                                                                                 ║
║ Status: ✓ Approved                                                                             ║
║ Classification: Internal — Confidential                                                        ║
║ Authors: NexusAsset Cloud Platform Engineering Team                                            ║
║ Reviewers: Security, Backend, Frontend, QA, Architecture                                       ║
║ Approved By: CTO / VP Platform Engineering / DevOps Lead                                       ║
║ Date: 2026-11-09                                                                               ║
║ Supersedes: —                                                                                  ║
║ Related: DOC-00, DOC-01, DOC-05, DOC-06, DOC-07, DOC-10, DOC-14, DOC-15, DOC-16              ║
║ Traceability: REQ-NFR-### ← CAP-080/081/082, ADR-### → DevOps Decisions                        ║
╚════════════════════════════════════════════════════════════════════════════════════════════════╝

# ═══════════════════════════════════════════════════════════════
# NexusAsset Cloud — DevOps & CI/CD (DOC-11)
# ═══════════════════════════════════════════════════════════════

> **Executive Summary** — This document defines the DevOps and CI/CD strategy for NexusAsset Cloud. The pipeline uses **Jenkins LTS** for orchestration, **Ansible** for configuration management, **ArgoCD** for GitOps-based Kubernetes deployments, and **Terraform** for infrastructure provisioning. The strategy implements **progressive delivery** with canary deployments, **automated rollback** on SLO breach, **feature flags** for safe releases, and **comprehensive quality gates** at every stage. All pipelines are defined as code and version-controlled in the monorepo.

---

## Table of Contents

1. [DevOps Principles](#1-devops-principles)
2. [CI/CD Architecture Overview](#2-cicd-architecture-overview)
3. [Source Control Strategy](#3-source-control-strategy)
4. [Continuous Integration (CI)](#4-continuous-integration-ci)
5. [Continuous Deployment (CD)](#5-continuous-deployment-cd)
6. [GitOps with ArgoCD](#6-gitops-with-argocd)
7. [Infrastructure as Code (Terraform/Ansible)](#7-infrastructure-as-code-terraformansible)
8. [Progressive Delivery](#8-progressive-delivery)
9. [Rollback Strategies](#9-rollback-strategies)
10. [Feature Flags & Experimentation](#10-feature-flags--experimentation)
11. [Secrets Management in CI/CD](#11-secrets-management-in-cicd)
12. [Pipeline Quality Gates](#12-pipeline-quality-gates)
13. [Deployment Environments](#13-deployment-environments)
14. [Monitoring & Observability in CI/CD](#14-monitoring--observability-in-cicd)
15. [Disaster Recovery for CI/CD](#15-disaster-recovery-for-cicd)
16. [Diagrams](#16-diagrams)
17. [Tables](#17-tables)

---

## 1. DevOps Principles

| Principle ID | Principle | Implementation |
|--------------|-----------|----------------|
| **DEVOPS-001** | **Everything as Code** | Pipelines, infrastructure, config, policies all in Git |
| **DEVOPS-002** | **GitOps for Deployments** | ArgoCD syncs desired state from Git to clusters |
| **DEVOPS-003** | **Shift-Left Security** | SAST, DAST, SCA in CI; policy-as-code (OPA) |
| **DEVOPS-004** | **Progressive Delivery** | Canary → Blue/Green → Full rollout with automated rollback |
| **DEVOPS-005** | **Immutable Infrastructure** | No manual changes; replace, don't mutate |
| **DEVOPS-006** | **Observability-Driven** | Deployments emit metrics; SLO-based validation |
| **DEVOPS-007** | **Self-Service Platform** | Developers deploy via PR; no ticket-based ops |
| **DEVOPS-008** | **Fast Feedback** | CI < 10 min; CD < 30 min; quality gates automated |
| **DEVOPS-009** | **Blameless Postmortems** | Incident reviews focus on system, not people |
| **DEVOPS-010** | **Continuous Improvement** | Weekly retro on pipeline metrics; DORA metrics tracked |

---

## 2. CI/CD Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              CI/CD ARCHITECTURE                                                     │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                              SOURCE CONTROL (GitHub)                                        │   │
│  │  • Monorepo: backend/, frontend/, platform/, docs/                                         │   │
│  │  • Branch Strategy: trunk-based + release branches                                         │   │
│  │  • Protected Branches: main, develop, release/*                                            │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                           │                                                         │
│                    ┌──────────────────────┼──────────────────────┐                              │
│                    ▼                      ▼                      ▼                              │
│           ┌───────────────┐       ┌───────────────┐       ┌───────────────┐                   │
│           │   JENKINS     │       │   GITHUB      │       │   ARGOCD      │                   │
│           │   (CI/CD)     │       │   ACTIONS     │       │   (GitOps)    │                   │
│           ├───────────────┤       ├───────────────┤       ├───────────────┤                   │
│           │ • Build       │       │ • Lint/Format │       │ • Sync Apps   │                   │
│           │ • Test        │       │ • SAST/SCA    │       │ • Health Checks│                   │
│           │ • Package     │       │ • Dependency  │       │ • Rollback    │                   │
│           │ • Security    │       │   Scan        │       │ • Drift Detect │                   │
│           │ • Quality     │       │ • Preview     │       │ • Notifications│                   │
│           │   Gates       │       │   Deployments │       │               │                   │
│           └───────┬───────┘       └───────┬───────┘       └───────┬───────┘                   │
│                   │                       │                       │                           │
│                   ▼                       ▼                       ▼                           │
│           ┌─────────────────────────────────────────────────────────────────────────────────┐   │
│           │                        ARTIFACT REPOSITORY (Harbor/JFrog)                       │   │
│           │  • Docker Images (backend, frontend, workers)                                   │   │
│           │  • Helm Charts (platform, monitoring, databases)                                │   │
│           │  • SBOMs (CycloneDX), Signatures (cosign)                                       │   │
│           └─────────────────────────────────────────────────────────────────────────────────┘   │
│                   │                       │                       │                           │
│                   ▼                       ▼                       ▼                           │
│           ┌─────────────────────────────────────────────────────────────────────────────────┐   │
│           │                    DEPLOYMENT TARGETS                                           │   │
│           │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │   │
│           │  │  DEV        │  │  STAGING    │  │  CANARY     │  │  PRODUCTION │           │   │
│           │  │  (per PR)   │  │  (main)     │  │  (5%→100%)  │  │  (ArgoCD)   │           │   │
│           │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘           │   │
│           └─────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Source Control Strategy

### 3.1 Branch Strategy

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              BRANCH STRATEGY (Trunk-Based + Release Branches)                       │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  main ────────────────────────────────────────────────────────────────────────────────────────▶    │
│   │                                                                                                │
│   │  ┌────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │  │                         RELEASE BRANCHES                                                │   │
│   │  │  release/v1.0 ─────────────────────────────────────────────────────────────────────▶   │   │
│   │  │     │    Hotfixes only (cherry-pick from main)                                        │   │
│   │  │     ▼                                                                                  │   │
│   │  │  release/v1.1 ─────────────────────────────────────────────────────────────────────▶   │   │
│   │  │                                                                                       │   │
│   │  │  release/v2.0 ─────────────────────────────────────────────────────────────────────▶   │   │
│   │  │                                                                                       │   │
│   │  └────────────────────────────────────────────────────────────────────────────────────────┘   │
│   │                                                                                                │
│   ├─ feature/xyz (short-lived, < 2 days)                                                         │
│   │    │                                                                                         │
│   │    └─ PR → main (squash merge, delete branch)                                                │
│   │                                                                                                │
│   ├─ bugfix/abc (short-lived)                                                                    │
│   │    │                                                                                         │
│   │    └─ PR → main (squash merge)                                                               │
│   │                                                                                                │
│   └─ hotfix/critical (from release branch)                                                       │
│        │                                                                                         │
│        └─ PR → release/v1.x → cherry-pick to main                                                │
│                                                                                                     │
│  BRANCH PROTECTION RULES:                                                                        │
│  • main: Require PR review (2), CI pass, up-to-date, signed commits                             │
│  • release/*: Require PR review (1), CI pass, restricted pushes                                 │
│  • develop: Auto-deploy to dev environment                                                       │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Commit Conventions (Conventional Commits)

```bash
# Format: <type>(<scope>): <subject>
# Types: feat, fix, docs, style, refactor, perf, test, chore, build, ci, revert

# Examples:
feat(assets): add bulk import with CSV validation
fix(billing): correct proration calculation on downgrade
docs(api): update OpenAPI spec for asset relationships
refactor(accounts): extract MFA verification to domain service
perf(assets): add database index for asset search
test(lansweeper): add contract tests for webhook receiver
chore(deps): update Django to 5.2.1
ci(pipeline): add chaos experiment validation stage
```

### 3.3 Version Tagging

```bash
# Semantic Versioning: MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]
# Tags pushed to Git and Docker registry

v1.0.0          # GA release
v1.0.1          # Patch
v1.1.0          # Minor
v2.0.0          # Major (breaking)
v1.2.0-rc.1     # Release candidate
v1.2.0-beta.3   # Beta
v1.2.0-alpha.1  # Alpha
v1.2.0+build.123 # Build metadata
```

---

## 4. Continuous Integration (CI)

### 4.1 Jenkins Pipeline Structure

```groovy
// platform/jenkins/pipelines/backend-ci.groovy
@Library('nexusasset-pipeline-lib@main') _

pipeline {
    agent { label 'docker' }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        skipDefaultCheckout()
        ansiColor('xterm')
    }
    
    environment {
        DOCKER_REGISTRY = 'harbor.nexusasset.cloud'
        IMAGE_NAME = "nexusasset/backend"
        SONAR_TOKEN = credentials('sonar-token')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git fetch --tags'
            }
        }
        
        stage('Code Quality') {
            parallel {
                stage('Lint') {
                    steps {
                        sh 'cd backend && ruff check . --output-format=github'
                        sh 'cd backend && black --check .'
                        sh 'cd backend && mypy --strict'
                    }
                }
                stage('Format') {
                    steps {
                        sh 'cd backend && ruff format --check .'
                    }
                }
                stage('Type Check') {
                    steps {
                        sh 'cd backend && mypy --strict --html-report mypy-report'
                    }
                }
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh '''
                    cd backend
                    pytest tests/unit \
                        -n 50 \
                        --cov=nexusasset \
                        --cov-fail-under=90 \
                        --cov-report=xml \
                        --cov-report=term-missing \
                        --junitxml=test-results/unit.xml \
                        -x
                '''
            }
            post {
                always {
                    publishTestResults testResultsPattern: 'backend/test-results/*.xml'
                    publishCoverage adapters: [coberturaAdapter('backend/coverage.xml')]
                }
            }
        }
        
        stage('Integration Tests') {
            steps {
                sh '''
                    cd backend
                    pytest tests/integration \
                        -n 10 \
                        --junitxml=test-results/integration.xml \
                        -x
                '''
            }
            post {
                always {
                    publishTestResults testResultsPattern: 'backend/test-results/integration.xml'
                }
            }
        }
        
        stage('Contract Tests') {
            steps {
                sh '''
                    cd backend
                    pytest tests/contracts/provider -x --junitxml=test-results/contract-provider.xml
                    pytest tests/contracts/events -x --junitxml=test-results/contract-events.xml
                '''
            }
        }
        
        stage('Security Scans') {
            parallel {
                stage('SAST') {
                    steps {
                        sh 'cd backend && bandit -r nexusasset -f json -o bandit-report.json'
                        sh 'cd backend && semgrep --config=auto --json=semgrep-report.json nexusasset'
                    }
                }
                stage('Dependency Scan') {
                    steps {
                        sh 'cd backend && pip-audit -r requirements.txt --format=json --output=pip-audit.json'
                        sh 'cd backend && safety check --json --output=safety-report.json'
                    }
                }
                stage('Container Scan') {
                    steps {
                        sh 'trivy image --severity HIGH,CRITICAL --format json --output trivy-report.json ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}'
                    }
                }
            }
        }
        
        stage('Build & Push Image') {
            when { branch 'main' }
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${imageTag}", "-f backend/Dockerfile backend")
                    docker.push("${DOCKER_REGISTRY}/${IMAGE_NAME}:${imageTag}")
                    
                    // Also tag as latest for main branch
                    docker.tag("${DOCKER_REGISTRY}/${IMAGE_NAME}:${imageTag}", "${DOCKER_REGISTRY}/${IMAGE_NAME}:latest")
                    docker.push("${DOCKER_REGISTRY}/${IMAGE_NAME}:latest")
                    
                    // Generate SBOM
                    sh "syft ${DOCKER_REGISTRY}/${IMAGE_NAME}:${imageTag} -o cyclonedx-json=sbom-${imageTag}.json"
                    sh "cosign sign --yes ${DOCKER_REGISTRY}/${IMAGE_NAME}:${imageTag}"
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                // Wait for SonarQube quality gate
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend channel: '#deployments', message: "✅ Backend CI passed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        failure {
            slackSend channel: '#deployments', message: "❌ Backend CI failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}
```

### 4.2 GitHub Actions for Fast Feedback

```yaml
# .github/workflows/ci-fast.yml
name: Fast CI (PR Validation)

on:
  pull_request:
    branches: [main, develop]
    types: [opened, synchronize, reopened]

jobs:
  lint-and-typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: 'pip'
      - name: Install dependencies
        run: |
          cd backend && pip install -r requirements-dev.txt
          cd ../frontend && npm ci
      - name: Backend Lint & Typecheck
        run: |
          cd backend
          ruff check . --output-format=github
          black --check .
          mypy --strict
      - name: Frontend Lint & Typecheck
        run: |
          cd frontend
          npm run lint
          npm run typecheck
      - name: Format Check
        run: |
          cd backend && ruff format --check .
          cd ../frontend && npm run format:check

  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: 'pip'
      - name: Run Unit Tests (Fast Subset)
        run: |
          cd backend
          pytest tests/unit \
            -n auto \
            --cov=nexusasset \
            --cov-fail-under=80 \
            -x \
            -q \
            --maxfail=5
        env:
          PYTHONPATH: /github/workspace/backend

  contract-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Provider Contract Tests
        run: |
          cd backend
          pytest tests/contracts/provider -x -q
      - name: Consumer Contract Tests
        run: |
          cd frontend
          npm run test:contract

  security-fast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: SAST (Semgrep)
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten
      - name: Dependency Scan
        run: |
          cd backend && pip-audit -r requirements.txt --format=json --output=pip-audit.json
          cd ../frontend && npm audit --audit-level=high

  preview-deploy:
    needs: [lint-and-typecheck, unit-tests, contract-tests, security-fast]
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Preview Environment
        run: |
          # Deploy to PR-specific namespace
          kubectl set image deployment/nxa-backend-preview \
            nxa-backend=harbor.nexusasset.cloud/nexusasset/backend:${{ github.sha }} \
            -n nxa-preview-${{ github.event.number }}
        env:
          KUBECONFIG: ${{ secrets.KUBECONFIG_STAGING }}
```

---

## 5. Continuous Deployment (CD)

### 5.1 Deployment Pipeline (Jenkins)

```groovy
// platform/jenkins/pipelines/backend-cd.groovy
@Library('nexusasset-pipeline-lib@main') _

pipeline {
    agent { label 'kubernetes' }
    
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag to deploy')
        string(name: 'ENVIRONMENT', defaultValue: 'staging', description: 'Target environment')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip pre-deploy tests')
        choice(name: 'STRATEGY', choices: ['rolling', 'canary', 'blue-green'], description: 'Deployment strategy')
    }
    
    environment {
        KUBECONFIG = credentials('kubeconfig-${params.ENVIRONMENT}')
        HELM_RELEASE = "nxa-backend"
        NAMESPACE = "nxa-backend"
    }
    
    stages {
        stage('Pre-Deploy Validation') {
            when { not { expression { return params.SKIP_TESTS } } }
            steps {
                sh '''
                    # Smoke tests against current version
                    kubectl run smoke-test-${BUILD_NUMBER} \
                        --image=harbor.nexusasset.cloud/nexusasset/backend:${IMAGE_TAG} \
                        --rm -i --restart=Never -- \
                        pytest tests/smoke -x -q
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    def deployScript = """
                        helm upgrade --install ${HELM_RELEASE} platform/helm/nxa-backend \
                            --namespace ${NAMESPACE} \
                            --set image.tag=${IMAGE_TAG} \
                            --set deployment.strategy=${params.STRATEGY} \
                            --set canary.enabled=${params.STRATEGY == 'canary'} \
                            --wait --timeout=10m
                    """
                    
                    if (params.STRATEGY == 'canary') {
                        deployScript += """
                            # Monitor canary for 10 minutes
                            sleep 600
                            # Check metrics
                            CANARY_ERROR_RATE=$(promql 'rate(http_requests_total{version="canary",status=~"5.."}[5m])')
                            CANARY_LATENCY=$(promql 'histogram_quantile(0.99, rate(http_request_duration_seconds_bucket{version="canary"}[5m]))')
                            
                            if (( $(echo "$CANARY_ERROR_RATE > 0.01" | bc -l) )) || \
                               (( $(echo "$CANARY_LATENCY > 1.0" | bc -l) )); then
                                echo "Canary metrics failed, rolling back"
                                helm rollback ${HELM_RELEASE} -n ${NAMESPACE}
                                exit 1
                            fi
                            
                            # Promote canary
                            helm upgrade ${HELM_RELEASE} platform/helm/nxa-backend \
                                --namespace ${NAMESPACE} \
                                --set canary.enabled=false \
                                --wait --timeout=5m
                        """
                    }
                    
                    sh deployScript
                }
            }
        }
        
        stage('Post-Deploy Validation') {
            steps {
                sh '''
                    # Health checks
                    kubectl wait --for=condition=Ready pod -l app=nxa-backend -n ${NAMESPACE} --timeout=300s
                    
                    # Smoke tests against new version
                    kubectl run smoke-test-post-${BUILD_NUMBER} \
                        --image=harbor.nexusasset.cloud/nexusasset/backend:${IMAGE_TAG} \
                        --rm -i --restart=Never -- \
                        pytest tests/smoke -x -q
                    
                    # Verify key endpoints
                    curl -f https://api.nexusasset.cloud/health/ready
                    curl -f https://api.nexusasset.cloud/api/v1/assets?limit=1
                '''
            }
        }
        
        stage('Notify & Record') {
            steps {
                sh '''
                    # Record deployment in audit log
                    curl -X POST https://api.nexusasset.cloud/api/v1/platform/deployments \
                        -H "Authorization: Bearer ${PLATFORM_API_TOKEN}" \
                        -d '{
                            "service": "nxa-backend",
                            "version": "'${IMAGE_TAG}'",
                            "environment": "'${ENVIRONMENT}'",
                            "strategy": "'${STRATEGY}'",
                            "deployed_by": "'${BUILD_USER}'",
                            "status": "SUCCESS"
                        }'
                    
                    # Update ArgoCD application (if using GitOps for config)
                    argocd app set nxa-backend-${ENVIRONMENT} --parameter image.tag=${IMAGE_TAG}
                '''
            }
        }
    }
    
    post {
        failure {
            script {
                // Automated rollback on failure
                if (params.STRATEGY != 'rolling') {
                    sh "helm rollback ${HELM_RELEASE} -n ${NAMESPACE}"
                }
                slackSend channel: '#deployments', message: "❌ Deployment failed, rolled back: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            }
        }
        success {
            slackSend channel: '#deployments', message: "✅ Deployment successful: ${env.JOB_NAME} #${env.BUILD_NUMBER} (${params.ENVIRONMENT})"
        }
    }
}
```

### 5.2 Frontend Deployment

```groovy
// platform/jenkins/pipelines/frontend-cd.groovy
stage('Deploy Frontend') {
    steps {
        script {
            // Next.js standalone output deployed via Helm
            def frontendDeploy = """
                helm upgrade --install nxa-frontend platform/helm/nxa-frontend \
                    --namespace nxa-frontend \
                    --set image.tag=${IMAGE_TAG} \
                    --set config.NEXT_PUBLIC_API_URL=https://api.nexusasset.cloud \
                    --set config.NEXT_PUBLIC_TENANT_ID=${TENANT_ID} \
                    --wait --timeout=10m
            """
            
            // Invalidate Cloudflare cache
            def cacheInvalidation = """
                curl -X POST "https://api.cloudflare.com/client/v4/zones/${CF_ZONE_ID}/purge_cache" \
                    -H "Authorization: Bearer ${CF_API_TOKEN}" \
                    -H "Content-Type: application/json" \
                    --data '{"purge_everything":true}'
            """
            
            sh frontendDeploy
            sh cacheInvalidation
        }
    }
}
```

---

## 6. GitOps with ArgoCD

### 6.1 ArgoCD Application Definitions

```yaml
# platform/argocd/applications/nxa-backend-prod.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nxa-backend-prod
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: nxa-production
  source:
    repoURL: https://github.com/nexusasset/nexusasset-cloud
    targetRevision: main
    path: platform/helm/nxa-backend
    helm:
      valueFiles:
        - values.yaml
        - values-prod.yaml
      parameters:
        - name: image.tag
          value: "latest"
        - name: replicaCount
          value: "9"
        - name: resources.limits.cpu
          value: "2000m"
        - name: resources.limits.memory
          value: "4Gi"
  destination:
    server: https://kubernetes.default.svc
    namespace: nxa-backend
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas  # HPA manages replicas
---
# platform/argocd/applications/nxa-frontend-prod.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nxa-frontend-prod
  namespace: argocd
spec:
  project: nxa-production
  source:
    repoURL: https://github.com/nexusasset/nexusasset-cloud
    targetRevision: main
    path: platform/helm/nxa-frontend
    helm:
      valueFiles:
        - values.yaml
        - values-prod.yaml
      parameters:
        - name: image.tag
          value: "latest"
  destination:
    server: https://kubernetes.default.svc
    namespace: nxa-frontend
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### 6.2 ArgoCD Project & RBAC

```yaml
# platform/argocd/projects/nxa-production.yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: nxa-production
  namespace: argocd
spec:
  description: Production applications for NexusAsset Cloud
  sourceRepos:
    - https://github.com/nexusasset/nexusasset-cloud
  destinations:
    - namespace: nxa-backend
      server: https://kubernetes.default.svc
    - namespace: nxa-frontend
      server: https://kubernetes.default.svc
    - namespace: nxa-worker
      server: https://kubernetes.default.svc
    - namespace: nxa-system
      server: https://kubernetes.default.svc
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
  namespaceResourceWhitelist:
    - group: '*'
      kind: '*'
  roles:
    - name: developer
      description: Can sync and view apps
      policies:
        - p, proj:nxa-production:developer, applications, get, nxa-production/*, allow
        - p, proj:nxa-production:developer, applications, sync, nxa-production/*, allow
    - name: operator
      description: Full access
      policies:
        - p, proj:nxa-production:operator, applications, *, nxa-production/*, allow
```

---

## 7. Infrastructure as Code (Terraform/Ansible)

### 7.1 Terraform Structure

```
platform/terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── terraform.tfvars
│       └── backend.tf
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── eks/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── node-groups.tf
│   ├── rds/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── elasticache/
│   ├── msk/
│   ├── s3/
│   ├── cloudfront/
│   ├── route53/
│   ├── acm/
│   ├── kms/
│   ├── iam/
│   └── monitoring/
├── providers.tf
├── versions.tf
└── backend.tf
```

### 7.2 Terraform CI/CD Pipeline

```groovy
// platform/jenkins/pipelines/terraform.groovy
pipeline {
    agent { label 'terraform' }
    
    parameters {
        choice(name: 'ACTION', choices: ['plan', 'apply', 'destroy'], description: 'Terraform action')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
        booleanParam(name: 'AUTO_APPROVE', defaultValue: false, description: 'Auto-apply without approval')
    }
    
    stages {
        stage('Validate') {
            steps {
                sh '''
                    cd platform/terraform/environments/${ENVIRONMENT}
                    terraform init -backend-config=backend.tf
                    terraform validate
                    terraform fmt -check -recursive
                '''
            }
        }
        
        stage('Plan') {
            steps {
                sh '''
                    cd platform/terraform/environments/${ENVIRONMENT}
                    terraform plan -out=tfplan -input=false
                '''
                // Upload plan as artifact
                archiveArtifacts artifacts: 'platform/terraform/environments/${ENVIRONMENT}/tfplan', fingerprint: true
            }
        }
        
        stage('Manual Approval') {
            when {
                expression { return params.ACTION == 'apply' && !params.AUTO_APPROVE }
            }
            steps {
                input message: "Apply Terraform changes for ${params.ENVIRONMENT}?", 
                      ok: 'Apply', 
                      submitter: 'platform-team'
            }
        }
        
        stage('Apply') {
            when {
                expression { return params.ACTION == 'apply' }
            }
            steps {
                sh '''
                    cd platform/terraform/environments/${ENVIRONMENT}
                    terraform apply -input=false tfplan
                '''
            }
        }
        
        stage('Destroy') {
            when {
                expression { return params.ACTION == 'destroy' }
            }
            steps {
                input message: "DESTROY ${params.ENVIRONMENT} environment? This is irreversible!", 
                      ok: 'Destroy', 
                      submitter: 'platform-lead'
                sh '''
                    cd platform/terraform/environments/${ENVIRONMENT}
                    terraform destroy -auto-approve
                '''
            }
        }
    }
}
```

### 7.3 Ansible for Configuration Management

```yaml
# platform/ansible/playbooks/configure-k8s-nodes.yml
---
- name: Configure Kubernetes Nodes
  hosts: k8s_nodes
  become: yes
  vars_files:
    - ../vars/common.yml
    - "../vars/{{ env }}.yml"
  
  pre_tasks:
    - name: Install required packages
      apt:
        name:
          - containerd
          - nfs-common
          - jq
        state: present
        update_cache: yes
  
  roles:
    - role: container-runtime
      tags: [containerd]
    - role: kubelet
      kubelet_version: "{{ kubernetes_version }}"
      tags: [kubernetes]
    - role: cilium
      tags: [cni]
    - role: monitoring-agent
      prometheus_url: "{{ prometheus_url }}"
      tags: [observability]
    - role: security-agent
      falco_enabled: true
      tags: [security]
    - role: log-forwarder
      loki_url: "{{ loki_url }}"
      tags: [logging]
    - role: backup-agent
      velero_enabled: true
      tags: [backup]
```

---

## 8. Progressive Delivery

### 8.1 Deployment Strategies

| Strategy | Use Case | Traffic Split | Validation | Rollback Time |
|----------|----------|---------------|------------|---------------|
| **Rolling** | Low-risk patches, config changes | 100% gradual | Health checks | ~2 min |
| **Canary** | New features, API changes | 5% → 25% → 100% | SLO metrics, error budget | ~1 min |
| **Blue/Green** | Major releases, DB migrations | Instant switch | Pre-deploy smoke tests | ~30 sec |
| **Feature Flag** | UI changes, risky logic | Per-user/tenant | Targeted rollout | Instant |

### 8.2 Canary Deployment with Flagger (Alternative to Helm)

```yaml
# platform/flagger/nxa-backend-canary.yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: nxa-backend
  namespace: nxa-backend
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nxa-backend
  service:
    port: 8000
    targetPort: 8000
    gateways:
      - nxa-traefik-public
    hosts:
      - api.nexusasset.cloud
  analysis:
    interval: 2m
    threshold: 5
    maxWeight: 100
    stepWeight: 10
    metrics:
      - name: request-success-rate
        thresholdRange:
          min: 99
        interval: 1m
      - name: request-duration-p99
        thresholdRange:
          max: 500
        interval: 1m
    webhooks:
      - name: load-test
        url: http://flagger-loadtester.test/
        timeout: 30s
        metadata:
          type: cmd
          cmd: "hey -z 1m -c 10 -host api.nexusasset.cloud http://nxa-backend.nxa-backend:8000/health/ready"
```

---

## 9. Rollback Strategies

### 9.1 Rollback Types

| Rollback Type | Trigger | Execution Time | Data Impact |
|---------------|---------|----------------|-------------|
| **Helm Rollback** | Failed deployment, metric breach | < 30 sec | None (stateless) |
| **Database Migration Rollback** | Migration failure, data corruption | 5-30 min | Reverts schema + data |
| **ArgoCD Rollback** | GitOps drift, bad config | < 1 min | None |
| **Image Rollback** | Container vulnerability, bug | < 2 min | None |
| **Feature Flag Kill Switch** | Critical bug in feature | Instant | None |
| **DNS Failover** | Region outage | < 5 min | None |
| **Full Disaster Recovery** | Region loss, data corruption | < 4 hours | RPO < 1hr |

### 9.2 Automated Rollback Rules

```yaml
# platform/argocd/rollbacks/auto-rollback.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nxa-backend-prod
  annotations:
    argocd.argoproj.io/sync-options: PrunePropagationPolicy=foreground
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    retry:
      limit: 3
      backoff:
        duration: 10s
        factor: 2
        maxDuration: 5m
  # Custom health check for auto-rollback
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
---
# Custom controller for SLO-based rollback
# platform/controllers/slo-rollback-controller.py
class SLORollbackController:
    def __init__(self):
        self.prometheus = PrometheusConnect()
        self.argocd = ArgoCDClient()
    
    def check_and_rollback(self, app_name: str, namespace: str):
        # Check error budget consumption
        error_budget = self.get_error_budget(app_name)
        
        if error_budget < 0.02:  # Less than 2% budget remaining
            # Check if recent deployment caused it
            recent_deploy = self.get_recent_deployment(app_name)
            if recent_deploy and recent_deploy.age < 30 * 60:  # 30 min
                self.argocd.rollback(app_name, namespace)
                alert_oncall(f"Auto-rollback triggered for {app_name}: error budget exhausted")
```

---

## 10. Feature Flags & Experimentation

### 10.1 Feature Flag Integration in CI/CD

```groovy
// platform/jenkins/pipelines/feature-flag.groovy
stage('Feature Flag Management') {
    steps {
        script {
            // Create feature flag for new feature
            if (env.CHANGE_TITLE?.contains('[FEATURE]')) {
                def flagKey = env.CHANGE_TITLE.replaceAll('.*\\[FEATURE:(.+?)\\].*', '$1')
                
                sh """
                    curl -X POST https://api.nexusasset.cloud/api/v1/platform/feature-flags \
                        -H "Authorization: Bearer ${PLATFORM_API_TOKEN}" \
                        -d '{
                            "key": "${flagKey}",
                            "name": "${flagKey}",
                            "flag_type": "BOOLEAN",
                            "default_value": false,
                            "targeting_rules": [
                                {"attribute": "tenant_id", "operator": "IN", "values": ["tn_preview"]}
                            ],
                            "variants": [
                                {"key": "on", "value": true, "weight": 10},
                                {"key": "off", "value": false, "weight": 90}
                            ],
                            "is_active": true
                        }'
                """
            }
            
            // Cleanup feature flag after merge
            if (currentBuild.rawBuild.getAction(hudson.model.CauseAction).causeList[0].shortDescription.contains('merge')) {
                // Find and archive flags from this PR
            }
        }
    }
}
```

### 10.2 Experimentation Framework

```python
# backend/nexusasset/core/domain/services/experimentation.py
class ExperimentationService:
    @staticmethod
    def run_experiment(
        experiment_id: str,
        control_variant: str,
        treatment_variant: str,
        traffic_split: float,  # 0.0-1.0
        metrics: List[str],
        duration_days: int = 14
    ) -> Experiment:
        """Create and launch A/B experiment"""
        experiment = Experiment(
            id=experiment_id,
            control=control_variant,
            treatment=treatment_variant,
            traffic_split=traffic_split,
            metrics=metrics,
            start_date=utc_now(),
            end_date=utc_now() + timedelta(days=duration_days),
            status=ExperimentStatus.RUNNING
        )
        
        ExperimentRepository.save(experiment)
        EventPublisher.publish(ExperimentStartedEvent(experiment=experiment))
        return experiment
    
    @staticmethod
    def evaluate_experiment(experiment_id: str) -> ExperimentResult:
        """Statistical evaluation using Bayesian inference"""
        experiment = ExperimentRepository.get(experiment_id)
        
        results = {}
        for metric in experiment.metrics:
            control_data = MetricsQuery.get(metric, variant=experiment.control)
            treatment_data = MetricsQuery.get(metric, variant=experiment.treatment)
            
            # Bayesian A/B test
            posterior = bayesian_ab_test(control_data, treatment_data)
            
            results[metric] = ExperimentMetricResult(
                metric=metric,
                probability_treatment_better=posterior.prob_better,
                expected_lift=posterior.expected_lift,
                credible_interval=posterior.credible_interval,
                recommendation=posterior.recommendation
            )
        
        return ExperimentResult(
            experiment_id=experiment_id,
            results=results,
            evaluated_at=utc_now()
        )
```

---

## 11. Secrets Management in CI/CD

### 11.1 Jenkins Credentials

```groovy
// Jenkins Credentials Binding
withCredentials([
    string(credentialsId: 'docker-registry-password', variable: 'DOCKER_PASSWORD'),
    string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN'),
    string(credentialsId: 'kubeconfig-prod', variable: 'KUBECONFIG'),
    string(credentialsId: 'argocd-token', variable: 'ARGOCD_TOKEN'),
    string(credentialsId: 'platform-api-token', variable: 'PLATFORM_API_TOKEN'),
    string(credentialsId: 'cf-api-token', variable: 'CF_API_TOKEN'),
    sshUserPrivateKey(credentialsId: 'github-deploy-key', keyFileVariable: 'GITHUB_DEPLOY_KEY'),
]) {
    // Pipeline steps with secrets injected as env vars
}
```

### 11.2 GitHub Actions Secrets

```yaml
# .github/workflows/ci.yml (secrets referenced)
env:
  DOCKER_REGISTRY: harbor.nexusasset.cloud
  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  KUBECONFIG_STAGING: ${{ secrets.KUBECONFIG_STAGING }}
  KUBECONFIG_PROD: ${{ secrets.KUBECONFIG_PROD }}
  ARGOCD_TOKEN: ${{ secrets.ARGOCD_TOKEN }}
  PLATFORM_API_TOKEN: ${{ secrets.PLATFORM_API_TOKEN }}
  CF_API_TOKEN: ${{ secrets.CF_API_TOKEN }}
  SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
  LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

### 11.3 Vault Integration for Runtime Secrets

```yaml
# platform/ansible/roles/vault-agent/templates/vault-agent-config.hcl
pid_file = "/home/vault/pidfile"
auto_auth {
  method "kubernetes" {
    mount_path = "auth/kubernetes"
    config = {
      role = "nxa-backend"
    }
  }
  
  sink "file" {
    config = {
      path = "/home/vault/secrets"
    }
  }
}

cache {
  use_auto_auth_token = true
}

listener "tcp" {
  address = "127.0.0.1:8100"
  tls_disable = true
}
```

---

## 12. Pipeline Quality Gates

### 12.1 Quality Gate Definitions

| Gate | Stage | Criteria | Tool | Blocking |
|------|-------|----------|------|----------|
| **Code Style** | CI | 0 lint errors | ruff, black, eslint | Yes |
| **Type Safety** | CI | 0 type errors | mypy, tsc | Yes |
| **Unit Coverage** | CI | Domain ≥ 90%, App ≥ 85% | pytest-cov, vitest | Yes |
| **Integration Pass** | CI | 100% pass | pytest | Yes |
| **Contract Compliance** | CI | 0 failures | schemathesis, pact | Yes |
| **SAST** | CI | 0 Critical/High | semgrep, bandit | Yes |
| **SCA** | CI | 0 Critical/High vulns | pip-audit, npm audit | Yes |
| **Container Scan** | CI | 0 Critical/High | trivy | Yes |
| **SonarQube Quality Gate** | CI | Pass | SonarQube | Yes |
| **E2E Critical Paths** | CD | 100% pass | Playwright | Yes |
| **Performance Regression** | CD | p99 < 10% increase | k6 | Yes |
| **Canary Metrics** | CD | Error rate < 1%, p99 < 1s | Prometheus | Yes |
| **Accessibility** | CD | 0 violations | axe-core | Yes |

### 12.2 Quality Gate Implementation (Jenkins)

```groovy
// platform/jenkins/pipelines/quality-gates.groovy
def qualityGates = [
    ['Unit Coverage Domain', 'pytest --cov=nexusasset --cov-report=json', { it.coverage.domain.lines >= 90 }],
    ['Unit Coverage App', 'pytest --cov=nexusasset --cov-report=json', { it.coverage.application.lines >= 85 }],
    ['Integration Tests', 'pytest tests/integration --junitxml=report.xml', { it.passRate == 100 }],
    ['Contract Tests', 'pytest tests/contracts --junitxml=report.xml', { it.failures == 0 }],
    ['SAST', 'semgrep --json=report.json', { it.critical == 0 && it.high == 0 }],
    ['SCA', 'pip-audit --format=json', { it.vulnerabilities.critical == 0 && it.vulnerabilities.high == 0 }],
    ['Container Scan', 'trivy image --format=json', { it.vulnerabilities.critical == 0 && it.vulnerabilities.high == 0 }],
    ['SonarQube', 'sonar-scanner', { it.qualityGate.status == 'OK' }],
]

stage('Quality Gates') {
    steps {
        script {
            def allPassed = true
            qualityGates.each { gate ->
                def name = gate[0]
                def cmd = gate[1]
                def check = gate[2]
                
                try {
                    def result = sh(script: cmd, returnStdout: true).trim()
                    def parsed = readJSON text: result
                    if (!check(parsed)) {
                        echo "❌ ${name} FAILED"
                        allPassed = false
                    } else {
                        echo "✅ ${name} PASSED"
                    }
                } catch (e) {
                    echo "❌ ${name} ERROR: ${e.message}"
                    allPassed = false
                }
            }
            
            if (!allPassed) {
                currentBuild.result = 'FAILURE'
                error "Quality gates failed"
            }
        }
    }
}
```

---

## 13. Deployment Environments

| Environment | Purpose | Trigger | Infrastructure | Data | Access |
|-------------|---------|---------|----------------|------|--------|
| **Local** | Development | Manual | kind, Docker Compose | Fixtures | Developer |
| **Preview (PR)** | PR Validation | PR opened | K8s namespace per PR | Anonymized subset | Team |
| **Dev** | Integration | Push to `develop` | 1 cluster (2 AZ) | Shared dev DB | Team |
| **Staging** | Pre-prod | Merge to `main` | 1 cluster (3 AZ) | Anonymized prod | Team + QA |
| **Canary** | Progressive rollout | Tag/Release | Prod cluster (5%→100%) | Real (5% traffic) | Platform |
| **Production** | Live | ArgoCD sync | 3 regions (3 AZ each) | Real tenant data | Platform + On-call |

---

## 14. Monitoring & Observability in CI/CD

### 14.1 Deployment Metrics

```prometheus
# Deployment metrics emitted by pipeline
deployment_total{service="nxa-backend",environment="prod",strategy="canary",status="success"}
deployment_duration_seconds{service="nxa-backend",environment="prod",phase="deploy"}
deployment_rollback_total{service="nxa-backend",reason="slo_breach"}
quality_gate_total{gate="unit_coverage",status="pass"}
```

### 14.2 DORA Metrics Tracking

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Deployment Frequency** | > 1/day | Deployments per day |
| **Lead Time for Changes** | < 1 hour | Commit to production |
| **Mean Time to Recovery** | < 30 min | Incident to resolution |
| **Change Failure Rate** | < 5% | Failed deployments / Total |

```python
# platform/monitoring/dora-metrics.py
class DORAMetricsCollector:
    def collect_deployment_frequency(self) -> float:
        """Deployments per day in last 30 days"""
        deployments = DeploymentRepository.get_last_30_days()
        return len(deployments) / 30
    
    def collect_lead_time(self) -> float:
        """Average commit to deploy time in hours"""
        deployments = DeploymentRepository.get_last_30_days()
        lead_times = [(d.deployed_at - d.committed_at).total_seconds() / 3600 for d in deployments]
        return sum(lead_times) / len(lead_times) if lead_times else 0
    
    def collect_mttr(self) -> float:
        """Average incident resolution time in minutes"""
        incidents = IncidentRepository.get_resolved_last_30_days()
        mttrs = [(i.resolved_at - i.created_at).total_seconds() / 60 for i in incidents]
        return sum(mttrs) / len(mttrs) if mttrs else 0
    
    def collect_change_failure_rate(self) -> float:
        """Failed deployments / total deployments"""
        total = DeploymentRepository.count_last_30_days()
        failed = DeploymentRepository.count_failed_last_30_days()
        return failed / total if total > 0 else 0
```

---

## 15. Disaster Recovery for CI/CD

### 15.1 CI/CD System Backup

| Component | Backup Method | Frequency | RPO | RTO |
|-----------|---------------|-----------|-----|-----|
| **Jenkins Config** | Git (JCasC) | On change | 0 | < 15 min |
| **Jenkins Jobs/History** | S3 (thinBackup) | Daily | 24h | < 1 hour |
| **GitHub Repos** | GitHub (native) | Continuous | 0 | < 5 min |
| **ArgoCD Config** | Git (App of Apps) | On change | 0 | < 10 min |
| **Secrets (Vault)** | Raft snapshots | Daily | 24h | < 30 min |
| **Terraform State** | S3 (versioned) | On apply | 0 | < 10 min |

### 15.2 CI/CD Recovery Procedures

```markdown
# platform/runbooks/cicd-disaster-recovery.md

## Jenkins Recovery
1. Provision new Jenkins controller from JCasC config in Git
2. Restore job configs from Git (JCasC) or thinBackup from S3
3. Reconfigure credentials from Vault
4. Verify agents connect
5. Run test pipeline

## ArgoCD Recovery
1. Reinstall ArgoCD operator in cluster
2. Apply AppProject and Application manifests from Git
3. ArgoCD will sync desired state automatically
4. Verify all applications Healthy

## GitHub Actions Recovery
1. No recovery needed (GitHub managed)
2. Verify secrets in organization settings
3. Re-run failed workflows

## Terraform State Recovery
1. Restore from S3 versioning (latest good state)
2. Run `terraform init` and `terraform plan` to verify
3. Apply if needed
```

---

## 16. Diagrams

| Diagram ID | Title | Type | Location | Description |
|------------|-------|------|----------|-------------|
| **DIA-115** | CI/CD Architecture | Component | Section 2 | Source → Jenkins/GHA → Artifacts → ArgoCD → Clusters |
| **DIA-116** | Branch Strategy | Flowchart | Section 3.1 | Trunk-based with release branches |
| **DIA-117** | Jenkins CI Pipeline | Flowchart | Section 4.1 | Stages: Checkout → Quality → Test → Security → Build → Quality Gate |
| **DIA-118** | GitHub Actions Fast CI | Flowchart | Section 4.2 | PR → Lint → Test → Contract → Security → Preview |
| **DIA-119** | CD Pipeline | Flowchart | Section 5.1 | Validate → Deploy (Strategy) → Post-Validate → Notify |
| **DIA-120** | GitOps with ArgoCD | Component | Section 6 | Git → ArgoCD → K8s → Health → Sync |
| **DIA-121** | Progressive Delivery | Flowchart | Section 8 | Strategy Selection → Canary/Blue-Green/Flag → Validate → Promote/Rollback |
| **DIA-122** | Rollback Strategies | Decision Tree | Section 9 | Trigger → Type → Execution → Validation |
| **DIA-123** | Feature Flag Lifecycle | State Diagram | Section 10 | Create → Target → Rollout → Evaluate → Archive |
| **DIA-124** | Terraform Pipeline | Flowchart | Section 7.2 | Validate → Plan → Approve → Apply |
| **DIA-125** | Quality Gates | Table | Section 12.1 | Gate → Criteria → Tool → Blocking |
| **DIA-126** | DORA Metrics | Component | Section 14.2 | Collection → Calculation → Dashboard |

---

## 17. Tables

| Table ID | Title | Location | Row Count |
|----------|-------|----------|-----------|
| **TBL-219** | DevOps Principles | Section 1 | 10 |
| **TBL-220** | Branch Protection Rules | Section 3.1 | 3 |
| **TBL-221** | Commit Types | Section 3.2 | 10 |
| **TBL-222** | Version Tagging | Section 3.3 | 7 |
| **TBL-223** | Deployment Strategies | Section 8.1 | 4 |
| **TBL-224** | Rollback Types | Section 9.1 | 7 |
| **TBL-225** | Quality Gates | Section 12.1 | 14 |
| **TBL-226** | Deployment Environments | Section 13 | 6 |
| **TBL-227** | DORA Metrics | Section 14.2 | 4 |
| **TBL-228** | CI/CD Backup Strategy | Section 15.1 | 6 |
| **TBL-229** | Diagram Index | Section 16 | 12 |
| **TBL-230** | Table Index | Section 17 | 11 |

---

---

**End of DOC-11 — NexusAsset Cloud DevOps & CI/CD**

*Document Control: Version 1.0.0 | Classification: Internal — Confidential | Next Review: 2026-12-09*

*For questions or updates: docs@nexusasset.cloud | Platform Team: platform@nexusasset.cloud*