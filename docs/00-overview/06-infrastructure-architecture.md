╔═════════════════════════════════════════════════════════════════════════════════════════════════╗
║ Document ID: DOC-06                                                                            ║
║ Title: Infrastructure Architecture — Platform, Data, Messaging, Security, Observability        ║
║ Version: 1.0.0                                                                                 ║
║ Status: ✓ Approved                                                                             ║
║ Classification: Internal — Confidential                                                        ║
║ Authors: NexusAsset Cloud Platform Engineering Team                                            ║
║ Reviewers: Security, SRE, Network, Database, Compliance                                        ║
║ Approved By: CTO / VP Platform Engineering / CISO                                              ║
║ Date: 2026-11-09                                                                               ║
║ Supersedes: —                                                                                  ║
║ Related: DOC-00, DOC-01, DOC-02, DOC-03, DOC-04, DOC-05, DOC-07, DOC-11, DOC-12, DOC-13       ║
║ Traceability: REQ-NFR-### ← CAP-###/SLO-###, ADR-### → Infrastructure Decisions, DIA-### → Diagrams ║
╚════════════════════════════════════════════════════════════════════════════════════════════════╝

# ═══════════════════════════════════════════════════════════════
# NexusAsset Cloud — Infrastructure Architecture (DOC-06)
# ═══════════════════════════════════════════════════════════════

> **Executive Summary** — This document defines the infrastructure architecture for NexusAsset Cloud, covering the platform topology (Kubernetes, networking, edge), data layer (PostgreSQL, Redis, object storage), messaging (Kafka, RabbitMQ), security infrastructure (Vault, mTLS, OPA), and observability stack (Prometheus, Grafana, Loki, Tempo, OpenTelemetry). The architecture targets **multi-region, multi-AZ deployment** with **tenant isolation at every layer**, **GitOps-driven operations**, and **SLO-backed reliability**. All infrastructure is defined as code (Terraform/Ansible) and deployed via Jenkins/ArgoCD pipelines (DOC-11).

---

## Table of Contents

1. [Platform Topology](#1-platform-topology)
2. [Kubernetes Cluster Architecture](#2-kubernetes-cluster-architecture)
3. [Networking & Edge (Traefik, mTLS, DNS)](#3-networking--edge-traefik-mtls-dns)
4. [Data Layer](#4-data-layer)
5. [Messaging & Event Streaming](#5-messaging--event-streaming)
6. [Security Infrastructure](#6-security-infrastructure)
7. [Observability Stack](#7-observability-stack)
8. [Secrets & Configuration Management](#8-secrets--configuration-management)
9. [Backup, Disaster Recovery & Business Continuity](#9-backup-disaster-recovery--business-continuity)
10. [Capacity Planning & Autoscaling](#10-capacity-planning--autoscaling)
11. [Infrastructure as Code (Terraform/Ansible)](#11-infrastructure-as-code-terraformansible)
12. [Environment Strategy](#12-environment-strategy)
13. [Diagrams](#13-diagrams)
14. [Tables](#14-tables)

---

## 1. Platform Topology

### 1.1 High-Level Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              NEXUSASSET CLOUD PLATFORM TOPOLOGY                                     │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                              EDGE LAYER (Global)                                            │   │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         │   │
│  │  │ Cloudflare CDN  │  │  Cloudflare WAF │  │  Cloudflare     │  │  Cloudflare     │         │   │
│  │  │  (Static Assets)│  │  (L7 Protection)│  │  Load Balancing │  │  DNS (nexusasset│         │   │
│  │  │                 │  │                 │  │  (Geo-routing)  │  │  .cloud)        │         │   │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘  └─────────────────┘         │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                           │                                                         │
│                                           ▼                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                          REGION: eu-west-1 (Primary)                                        │   │
│  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │   │
│  │  │                        TRAEFIK EDGE (nxa-traefik-public)                            │   │   │
│  │  │  • TLS Termination (Let's Encrypt via nxa-le)                                       │   │   │
│  │  │  • mTLS to Services (SPIFFE)                                                        │   │   │
│  │  │  • Rate Limiting, Auth Forwarding, Canary Routing                                   │   │   │
│  │  └─────────────────────────────────────────────────────────────────────────────────────┘   │   │
│  │                                           │                                                 │   │
│  │           ┌───────────────────┬───────────┴───────────┬───────────────────┐              │   │
│  │           ▼                   ▼                       ▼                   ▼              │   │
│  │  ┌───────────────┐   ┌───────────────┐         ┌───────────────┐   ┌───────────────┐    │   │
│  │  │  AZ-A         │   │  AZ-B         │         │  AZ-C         │   │  MANAGED      │    │   │
│  │  │  (Primary)    │   │  (Secondary)  │         │  (Tertiary)   │   │  SERVICES     │    │   │
│  │  ├───────────────┤   ├───────────────┤         ├───────────────┤   ├───────────────┤    │   │
│  │  │ nxa-backend   │   │ nxa-backend   │         │ nxa-backend   │   │ RDS           │    │   │
│  │  │ (3 replicas)  │   │ (3 replicas)  │         │ (3 replicas)  │   │ PostgreSQL 18 │    │   │
│  │  │ nxa-frontend  │   │ nxa-frontend  │         │ nxa-frontend  │   │ (Primary)     │    │   │
│  │  │ (2 replicas)  │   │ (2 replicas)  │         │ (2 replicas)  │   │               │    │   │
│  │  │ nxa-worker    │   │ nxa-worker    │         │ nxa-worker    │   │ ElastiCache   │    │   │
│  │  │ (5 replicas)  │   │ (3 replicas)  │         │ (3 replicas)  │   │ Redis 8       │    │   │
│  │  │               │   │               │         │               │   │ (Cluster)     │    │   │
│  │  │ PgBouncer     │   │ PgBouncer     │         │ PgBouncer     │   │               │    │   │
│  │  │ (Sidecar)     │   │ (Sidecar)     │         │ (Sidecar)     │   │ MSK Kafka     │    │   │
│  │  │               │   │               │         │               │   │ (3 broker AZ) │    │   │
│  │  └───────────────┘   └───────────────┘         └───────────────┘   │ S3/MinIO      │    │   │
│  │                                                                       │ OpenSearch    │    │   │
│  │                                                                       └───────────────┘    │   │
│  │                                                                                       │   │
│  └───────────────────────────────────────────────────────────────────────────────────────┘   │
│                                           │                                                         │
│                    ┌──────────────────────┼──────────────────────┐                              │
│                    ▼                      ▼                      ▼                              │
│           ┌───────────────┐       ┌───────────────┐       ┌───────────────┐                   │
│           │ REGION:       │       │ REGION:       │       │ REGION:       │                   │
│           │ us-east-1     │       │ eu-central-1  │       │ ap-southeast-1│                   │
│           │ (DR/Secondary)│       │ (EU Data      │       │ (APAC)        │                   │
│           │               │       │  Residency)   │       │               │                   │
│           │ Async Replica │       │ Sync Replica  │       │ Async Replica │                   │
│           │ RPO < 1hr     │       │ RPO = 0       │       │ RPO < 1hr     │                   │
│           │ RTO < 4hr     │       │ RTO < 30min   │       │ RTO < 4hr     │                   │
│           └───────────────┘       └───────────────┘       └───────────────┘                   │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Service Inventory

| Service | Type | Replicas (per AZ) | Resources (per pod) | Scaling |
|---------|------|-------------------|---------------------|---------|
| **nxa-backend** | Django 5.2 + DRF | 3 (AZ-A), 3 (AZ-B), 3 (AZ-C) | 2 CPU, 4 GiB, 10 GiB ephemeral | HPA: CPU>70%, custom metrics (queue depth) |
| **nxa-frontend** | Next.js 16 (BFF) | 2 (AZ-A), 2 (AZ-B), 2 (AZ-C) | 1 CPU, 2 GiB, 5 GiB ephemeral | HPA: CPU>60%, request latency |
| **nxa-worker** | Celery 5.3 | 5 (AZ-A), 3 (AZ-B), 3 (AZ-C) | 2 CPU, 4 GiB, 20 GiB ephemeral | KEDA: queue depth > 100 |
| **nxa-traefik-public** | Traefik v3 | 3 (per AZ) | 1 CPU, 1 GiB | HPA: request rate |
| **nxa-le** | Cert-manager | 1 (per cluster) | 0.5 CPU, 512 MiB | Static |
| **nxa-db** | PostgreSQL 18 (RDS) | 1 Primary + 2 Replicas | db.r6g.2xlarge (8 vCPU, 64 GiB) | Vertical + Read Replicas |
| **nxa-redis** | Redis 8 (ElastiCache) | Cluster: 3 shards × 2 replicas | cache.r6g.xlarge (4 vCPU, 32 GiB) | Add shards |
| **nxa-kafka** | MSK Kafka | 3 brokers (1 per AZ) | kafka.m7g.large (2 vCPU, 8 GiB) | Add brokers |
| **nxa-rabbitmq** | RabbitMQ (Optional) | 3 (quorum queue) | 2 CPU, 4 GiB | Add nodes |
| **nxa-minio** | MinIO (S3) | 4 nodes (distributed) | 4 CPU, 16 GiB, 500 GiB | Add nodes |

---

## 2. Kubernetes Cluster Architecture

### 2.1 Cluster Specification

```yaml
# platform/kubernetes/cluster-spec.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-spec
  namespace: kube-system
data:
  cluster-name: "nxa-prod-eu-west-1"
  kubernetes-version: "1.29"
  cni: "cilium"                    # eBPF-based, NetworkPolicy, Hubble observability
  csi-drivers:
    - ebs-csi-driver               # AWS EBS volumes
    - efs-csi-driver               # AWS EFS for shared storage
  ingress-controller: "traefik"    # Traefik v3 via Helm
  service-mesh: "none"             # mTLS via Cilium + SPIFFE (no Istio/Linkerd)
  node-groups:
    - name: "general"
      instance-type: "m6i.xlarge"  # 4 vCPU, 16 GiB
      min-size: 3
      max-size: 20
      labels:
        workload: "general"
      taints: []
    - name: "worker"
      instance-type: "m6i.2xlarge" # 8 vCPU, 32 GiB
      min-size: 3
      max-size: 30
      labels:
        workload: "batch"
      taints:
        - key: "workload"
          value: "batch"
          effect: "NoSchedule"
    - name: "memory-optimized"
      instance-type: "r6i.2xlarge" # 8 vCPU, 64 GiB
      min-size: 2
      max-size: 10
      labels:
        workload: "database"
      taints:
        - key: "workload"
          value: "database"
          effect: "NoSchedule"
```

### 2.2 Namespace Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              KUBERNETES NAMESPACE ARCHITECTURE                                      │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                        PLATFORM NAMESPACES (Cluster-scoped)                                 │   │
│  ├─────────────────────────────────────────────────────────────────────────────────────────────┤   │
│  │  kube-system          │ Core Kubernetes components                                          │   │
│  │  cilium               │ Cilium CNI, Hubble                                                  │   │
│  │  cert-manager         │ TLS certificate management (nxa-le)                                 │   │
│  │  monitoring           │ Prometheus, Grafana, Alertmanager, Loki, Tempo                     │   │
│  │  logging              │ Fluent Bit, Loki (if separate)                                      │   │
│  │  argocd               │ ArgoCD GitOps operator                                              │   │
│  │  kafka                │ Strimzi Kafka operator (if self-managed)                            │   │
│  │  rabbitmq             │ RabbitMQ operator (if self-managed)                                 │   │
│  │  minio                │ MinIO operator (if self-managed)                                    │   │
│  │  vault                │ HashiCorp Vault (if self-managed)                                   │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                        TENANT NAMESPACES (Per-tenant isolation)                             │   │
│  ├─────────────────────────────────────────────────────────────────────────────────────────────┤   │
│  │  nxa-tenant-{tenant_id}                                                    (1 per tenant)    │   │
│  │  ├─ NetworkPolicy: deny all ingress/egress except:                                          │   │
│  │  │  • Ingress from nxa-traefik-public (label: app=nxa-traefik-public)                     │   │
│  │  │  • Egress to nxa-db, nxa-redis, nxa-kafka, nxa-minio (via ServiceMesh/NetworkPolicy)  │   │
│  │  │  • Egress to internet (for webhooks, cloud provider APIs) via egress gateway          │   │
│  │  ├─ ResourceQuota: CPU, memory, pods, services, PVCs per tenant plan                      │   │
│  │  ├─ LimitRange: default container limits                                                    │   │
│  │  ├─ PodSecurityPolicy: restricted (no privileged, runAsNonRoot, readOnlyRootFs)           │   │
│  │  └─ ServiceAccount: tenant-scoped (IRSA for AWS, Workload Identity for GCP)              │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                        SHARED SERVICE NAMESPACES                                             │   │
│  ├─────────────────────────────────────────────────────────────────────────────────────────────┤   │
│  │  nxa-backend          │ Backend pods (shared across tenants via RLS, not namespace)        │   │
│  │  nxa-frontend         │ Frontend pods (shared)                                              │   │
│  │  nxa-worker           │ Celery workers (shared)                                             │   │
│  │  nxa-system           │ Platform services: nxa-core, feature flags, audit log writer       │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
│  NOTE: We use SHARED backend/worker namespaces with RLS for tenant isolation                     │
│  (not per-tenant namespaces for apps) to reduce operational overhead.                            │
│  Per-tenant namespaces used ONLY for: custom workloads, data-plane isolation, compliance.       │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Workload Deployment Patterns

| Workload | Deployment Strategy | Pod Disruption Budget | Rolling Update |
|----------|---------------------|----------------------|----------------|
| **nxa-backend** | RollingUpdate (maxSurge=25%, maxUnavailable=0) | minAvailable=2 | Partitioned rollout (10% → 50% → 100%) |
| **nxa-frontend** | RollingUpdate (maxSurge=50%, maxUnavailable=0) | minAvailable=1 | Blue/Green via ArgoCD |
| **nxa-worker** | Recreate (for stateful tasks) / RollingUpdate | minAvailable=50% | Rolling with drain |
| **nxa-traefik** | RollingUpdate (maxSurge=1, maxUnavailable=0) | minAvailable=2 | Standard |

---

## 3. Networking & Edge (Traefik, mTLS, DNS)

### 3.1 Traefik Configuration (nxa-traefik-public)

```yaml
# platform/kubernetes/traefik/values.yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: nxa-api
  namespace: nxa-system
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`api.nexusasset.cloud`) && PathPrefix(`/api`)
      kind: Rule
      services:
        - name: nxa-backend
          port: 8000
          namespace: nxa-backend
      middlewares:
        - name: rate-limit-tenant
          namespace: nxa-system
        - name: auth-forward
          namespace: nxa-system
        - name: correlation-id
          namespace: nxa-system
        - name: tenant-context
          namespace: nxa-system
---
# Middleware: Rate Limiting per Tenant
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: rate-limit-tenant
  namespace: nxa-system
spec:
  rateLimit:
    average: 1000      # requests/second per tenant
    burst: 2000
    sourceCriterion:
      ipStrategy: {}   # Use X-Forwarded-For
      requestHeaderName: "X-Tenant-ID"  # Rate limit by tenant
---
# Middleware: Authentication Forwarding to nxa-core
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: auth-forward
  namespace: nxa-system
spec:
  forwardAuth:
    address: "http://nxa-core.nxa-system:8080/auth/validate"
    authResponseHeaders:
      - "X-User-ID"
      - "X-Tenant-ID"
      - "X-Roles"
      - "X-Permissions"
    trustForwardHeader: true
---
# Middleware: Correlation ID Injection
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: correlation-id
  namespace: nxa-system
spec:
  plugin:
    correlation-id:
      headerName: "X-Request-ID"
      generator: "uuid"
      preserveExisting: true
```

### 3.2 mTLS Configuration (Cilium + SPIFFE)

```yaml
# platform/kubernetes/cilium/mtls-config.yaml
apiVersion: cilium.io/v2
kind: CiliumClusterwideNetworkPolicy
metadata:
  name: mtls-enforcement
spec:
  endpointSelector:
    matchLabels:
      app.kubernetes.io/part-of: nexusasset
  ingress:
    - fromEndpoints:
        - matchLabels:
            app.kubernetes.io/part-of: nexusasset
      toPorts:
        - ports:
            - port: "8000"
              protocol: TCP
          rules:
            http:
              - method: "GET"
                path: "/health/**"
          tls:
            mode: "require"  # Require mTLS for all service-to-service
            spiFFE:
              trustDomain: "nexusasset.cloud"
---
# SPIRE Agent DaemonSet for workload identity
apiVersion: spire.spiffe.io/v1
kind: ClusterSPIFFEID
metadata:
  name: nxa-workload-identity
spec:
  trustDomain: nexusasset.cloud
  namespaceSelector:
    matchLabels:
      workload: "true"
  podSelector:
    matchLabels:
      app.kubernetes.io/part-of: nexusasset
  federatesWith:
    - trustDomain: "partner.example.com"
      bundleEndpoint: "https://spire.partner.example.com/bundle"
```

### 3.3 DNS & Domain Strategy

| Domain | Purpose | Certificate | Routing |
|--------|---------|-------------|---------|
| `app.nexusasset.cloud` | Frontend (Next.js) | Cloudflare + Let's Encrypt | Cloudflare → Traefik → nxa-frontend |
| `api.nexusasset.cloud` | Public API (Django) | Cloudflare + Let's Encrypt | Cloudflare → Traefik → nxa-backend |
| `*.app.nexusasset.cloud` | Tenant custom domains | Let's Encrypt (per tenant) | Cloudflare → Traefik → nxa-frontend (tenant context) |
| `admin.nexusasset.cloud` | Platform admin UI | Cloudflare + Let's Encrypt | Cloudflare → Traefik → nxa-frontend (admin) |
| `docs.nexusasset.cloud` | Developer portal | Cloudflare + Let's Encrypt | Cloudflare → Traefik → nxa-core (portal) |
| `internal.nexusasset.cloud` | Internal services | Self-signed (mTLS) | Cluster-internal only |

---

## 4. Data Layer

### 4.1 PostgreSQL 18 Architecture (nxa-db)

```yaml
# platform/terraform/aws/rds/main.tf
resource "aws_db_instance" "nxa_primary" {
  identifier             = "nxa-prod-primary"
  engine                 = "postgres"
  engine_version         = "18.0"
  instance_class         = "db.r6g.2xlarge"
  allocated_storage      = 500
  max_allocated_storage  = 2000
  storage_encrypted      = true
  kms_key_id             = aws_kms_key.nxa_db.arn
  
  db_name                = "nexusasset"
  username               = "nxa_admin"
  password               = random_password.db_password.result
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.nxa.name
  
  backup_retention_period = 35
  backup_window          = "03:00-04:00"
  maintenance_window     = "sun:04:00-sun:05:00"
  
  deletion_protection    = true
  skip_final_snapshot    = false
  
  # Read replicas for scaling reads
  replica_count          = 2
  replica_instance_class = "db.r6g.xlarge"
  
  # Performance Insights
  performance_insights_enabled = true
  performance_insights_kms_key_id = aws_kms_key.nxa_db.arn
  
  # Log exports
  enabled_cloudwatch_logs_exports = ["postgresql", "upgrade"]
  
  # Parameter group for RLS optimization
  parameter_group_name   = aws_db_parameter_group.nxa_optimized.name
}
```

### 4.2 Row-Level Security (RLS) Implementation

```sql
-- backend/nexusasset/shared/kernel/infrastructure/database/migrations/0002_rls_policies.sql
-- Enable RLS on all tenant-scoped tables
ALTER TABLE assets ENABLE ROW LEVEL SECURITY;
ALTER TABLE ci_relationships ENABLE ROW LEVEL SECURITY;
ALTER TABLE maintenance_windows ENABLE ROW LEVEL SECURITY;
ALTER TABLE subscriptions ENABLE ROW LEVEL SECURITY;
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;
-- ... all tenant tables

-- Create RLS policy using session variable
CREATE POLICY tenant_isolation ON assets
  USING (tenant_id = current_setting('app.tenant_id')::uuid);

-- Bypass for platform admins (superuser role)
CREATE POLICY platform_admin_bypass ON assets
  USING (current_user = 'nxa_platform_admin');

-- Function to set tenant context (called by middleware)
CREATE OR REPLACE FUNCTION set_tenant_context(tenant_id uuid) RETURNS void AS $$
BEGIN
  PERFORM set_config('app.tenant_id', tenant_id::text, false);
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Index for RLS performance
CREATE INDEX idx_assets_tenant_id ON assets (tenant_id);
CREATE INDEX idx_assets_tenant_state ON assets (tenant_id, state);
CREATE INDEX idx_ci_relationships_tenant ON ci_relationships (tenant_id);
```

### 4.3 PgBouncer Configuration (Connection Pooling)

```ini
# platform/kubernetes/pgbouncer/pgbouncer.ini
[databases]
nexusasset = host=nxa-prod-primary.cluster-xyz.eu-west-1.rds.amazonaws.com port=5432 dbname=nexusasset

[pgbouncer]
pool_mode = transaction
max_client_conn = 10000
default_pool_size = 100
min_pool_size = 20
reserve_pool_size = 10
reserve_pool_timeout = 5
max_db_connections = 500
max_user_connections = 100

# TLS
client_tls_sslmode = require
server_tls_sslmode = require

# Stats
stats_period = 60
```

### 4.4 Redis 8 Cluster (nxa-redis)

```yaml
# platform/terraform/aws/elasticache/main.tf
resource "aws_elasticache_replication_group" "nxa_redis" {
  replication_group_id       = "nxa-prod-redis"
  engine                     = "redis"
  engine_version             = "8.0"
  node_type                  = "cache.r6g.xlarge"
  number_of_shards           = 3
  replicas_per_node_group    = 1
  automatic_failover_enabled = true
  multi_az_enabled           = true
  
  # Encryption
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  auth_token                 = random_password.redis_auth.result
  
  # Cluster mode
  cluster_mode_enabled       = true
  
  # Snapshots
  snapshot_retention_limit   = 7
  snapshot_window            = "03:00-04:00"
  
  # Maintenance
  maintenance_window         = "sun:05:00-sun:06:00"
  
  # Security groups
  security_group_ids         = [aws_security_group.elasticache.id]
  subnet_group_name          = aws_elasticache_subnet_group.nxa.name
}
```

**Redis Usage Patterns**:
| Use Case | Key Pattern | TTL | Eviction Policy |
|----------|-------------|-----|-----------------|
| Tenant config | `tenant:{tid}:config` | 5 min | volatile-lru |
| User permissions | `user:{uid}:perms` | 1 min | volatile-lru |
| Asset lookups | `asset:{aid}:data` | 30 sec | volatile-lru |
| Rate limiting | `ratelimit:{tid}:{endpoint}` | 1 sec | volatile-lru |
| Session data | `session:{sid}` | 30 min | volatile-ttl |
| Idempotency keys | `idem:{key}` | 24 hr | volatile-ttl |
| Celery results | `celery:{task_id}` | 1 hr | volatile-ttl |

### 4.5 Object Storage (MinIO / S3)

```yaml
# platform/kubernetes/minio/tenant.yaml
apiVersion: minio.min.io/v2
kind: Tenant
metadata:
  name: nxa-minio
  namespace: minio
spec:
  image: minio/minio:RELEASE.2024-11-09
  replicas: 4
  volumesPerServer: 4
  storageClassName: gp3
  resources:
    requests:
      memory: 16Gi
      cpu: "4"
    limits:
      memory: 16Gi
      cpu: "4"
  
  # Buckets per purpose
  buckets:
    - name: nxa-audit-evidence
      policy: |
        {"Versioning": {"Status": "Enabled"}, "Lifecycle": {"Rules": [{"ID": "archive", "Status": "Enabled", "Filter": {}, "Transitions": [{"Days": 90, "StorageClass": "GLACIER"}]}]}}
    - name: nxa-scan-artifacts
      policy: |
        {"Lifecycle": {"Rules": [{"ID": "expire", "Status": "Enabled", "Filter": {}, "Expiration": {"Days": 30}}]}}
    - name: nxa-backups
      policy: |
        {"Versioning": {"Status": "Enabled"}, "Lifecycle": {"Rules": [{"ID": "archive", "Status": "Enabled", "Filter": {}, "Transitions": [{"Days": 30, "StorageClass": "STANDARD_IA"}, {"Days": 365, "StorageClass": "GLACIER"}]}]}}
    - name: nxa-exports
      policy: |
        {"Lifecycle": {"Rules": [{"ID": "expire", "Status": "Enabled", "Filter": {}, "Expiration": {"Days": 7}}]}}
    - name: nxa-frontend-assets
      policy: |
        {"Versioning": {"Status": "Enabled"}, "Cors": [{"AllowedOrigins": ["*"], "AllowedMethods": ["GET"], "AllowedHeaders": ["*"]}]}
```

---

## 5. Messaging & Event Streaming

### 5.1 Kafka Cluster (MSK / Strimzi)

```yaml
# platform/kubernetes/kafka/kafka-cluster.yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: nxa-kafka
  namespace: kafka
spec:
  kafka:
    version: 3.7
    replicas: 3
    resources:
      requests:
        memory: 8Gi
        cpu: "2"
      limits:
        memory: 8Gi
        cpu: "2"
    config:
      num.partitions: 48
      default.replication.factor: 3
      min.insync.replicas: 2
      compression.type: zstd
      log.retention.hours: 168          # 7 days hot
      log.retention.check.interval.ms: 300000
      log.segment.bytes: 1073741824     # 1GB segments
      message.max.bytes: 1048588        # ~1MB
    storage:
      type: persistent-claim
      size: 500Gi
      class: gp3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
        authentication:
          type: tls
      - name: external
        port: 9094
        type: loadbalancer
        tls: true
        authentication:
          type: tls
  
  zookeeper:
    replicas: 3
    resources:
      requests:
        memory: 2Gi
        cpu: "1"
    storage:
      type: persistent-claim
      size: 100Gi
  
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

### 5.2 Topic Design

| Topic | Partitions | Retention | Compaction | Purpose |
|-------|------------|-----------|------------|---------|
| `nxa.audit.raw` | 48 | 90 days | No | Raw audit logs (hot) |
| `nxa.assets.discovered` | 24 | 30 days | No | AssetDiscovered events |
| `nxa.assets.reconciled` | 24 | 30 days | No | AssetReconciled events |
| `nxa.assets.state_changed` | 24 | 90 days | No | AssetStateChanged events |
| `nxa.tenants.lifecycle` | 12 | 365 days | Yes (key=tenant_id) | Tenant provisioned/suspended/offboarded |
| `nxa.billing.invoices` | 12 | 365 days | No | InvoiceGenerated/Paid events |
| `nxa.notifications.queue` | 24 | 7 days | No | Notification delivery queue |
| `nxa.lansweeper.sync` | 12 | 30 days | No | SyncJob events |
| `nxa.audit.findings` | 12 | 365 days | No | Compliance findings |
| `nxa.core.feature_flags` | 6 | 365 days | Yes (key=flag_key) | Feature flag updates |
| `nxa.core.background_jobs` | 24 | 7 days | No | Job lifecycle events |

### 5.3 Producer/Consumer Patterns

```python
# backend/nexusasset/shared/kernel/infrastructure/messaging/kafka_producer.py
class KafkaProducer:
    def __init__(self, bootstrap_servers: str, schema_registry: SchemaRegistry):
        self.producer = AIOKafkaProducer(
            bootstrap_servers=bootstrap_servers,
            compression_type="zstd",
            acks="all",
            max_in_flight_requests_per_connection=5,
            enable_idempotence=True,
            transactional_id="nxa-producer",
            key_serializer=lambda k: k.encode() if k else None,
            value_serializer=lambda v: v.model_dump_json().encode(),
        )
        self.schema_registry = schema_registry
    
    async def send(self, topic: str, event: IntegrationEvent) -> None:
        # Validate against schema registry
        schema_id = await self.schema_registry.get_schema_id(type(event).__name__)
        headers = [("schema_id", schema_id.to_bytes(4, "big"))]
        
        await self.producer.send_and_wait(
            topic=topic,
            key=event.correlation_id,
            value=event,
            headers=headers,
        )

# Consumer with exactly-once semantics
class KafkaConsumer:
    def __init__(self, topics: List[str], group_id: str, handler: EventHandler):
        self.consumer = AIOKafkaConsumer(
            *topics,
            bootstrap_servers=settings.KAFKA_BOOTSTRAP,
            group_id=group_id,
            enable_auto_commit=False,
            isolation_level="read_committed",
            value_deserializer=lambda m: json.loads(m.decode()),
        )
        self.handler = handler
    
    async def run(self) -> None:
        await self.consumer.start()
        async for msg in self.consumer:
            try:
                event = IntegrationEvent.parse_raw(msg.value)
                await self.handler.handle(event)
                await self.consumer.commit()
            except Exception as e:
                # Dead letter queue
                await self.dlq.send(msg, str(e))
                await self.consumer.commit()  # Skip poison pill
```

---

## 6. Security Infrastructure

### 6.1 HashiCorp Vault (Secrets Management)

```yaml
# platform/kubernetes/vault/values.yaml
global:
  enabled: true
  
server:
  resources:
    requests:
      memory: 2Gi
      cpu: "1"
    limits:
      memory: 4Gi
      cpu: "2"
  
  ha:
    enabled: true
    replicas: 3
    raft:
      enabled: true
      config: |
        cluster_name = "nxa-vault"
        storage "raft" {
          path = "/vault/data"
        }
  
  auditStorage:
    enabled: true
    size: 10Gi
  
  # Auto-unseal with AWS KMS
  seal:
    type: awskms
    config:
      region: eu-west-1
      kms_key_id: "{{ .Values.aws.kmsKeyId }}"
  
  # Injector for pod annotations
  injector:
    enabled: true
    resources:
      requests:
        memory: 256Mi
        cpu: "100m"
```

**Secret Paths**:
| Path | Purpose | Rotation |
|------|---------|----------|
| `secret/data/nxa/prod/database` | PostgreSQL credentials | 90 days |
| `secret/data/nxa/prod/redis` | Redis auth token | 90 days |
| `secret/data/nxa/prod/kafka` | Kafka SASL credentials | 90 days |
| `secret/data/nxa/prod/minio` | MinIO access/secret keys | 90 days |
| `secret/data/nxa/prod/stripe` | Stripe API keys | 90 days |
| `secret/data/nxa/prod/lansweeper` | Lansweeper PAT/OAuth secrets | 90 days |
| `secret/data/nxa/prod/smtp` | SendGrid/SMTP credentials | 90 days |
| `secret/data/nxa/prod/twilio` | Twilio credentials | 90 days |
| `secret/data/nxa/prod/jwt` | JWT signing keys (RS256) | 365 days |
| `secret/data/nxa/prod/tls` | TLS certificates (auto from cert-manager) | Auto |

### 6.2 External Secrets Operator (K8s Integration)

```yaml
# platform/kubernetes/external-secrets/externalsecret.yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: nxa-database-credentials
  namespace: nxa-backend
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: nxa-db-credentials
    creationPolicy: Owner
  data:
    - secretKey: username
      remoteRef:
        key: secret/data/nxa/prod/database
        property: username
    - secretKey: password
      remoteRef:
        key: secret/data/nxa/prod/database
        property: password
```

### 6.3 Network Policies (Cilium)

```yaml
# platform/kubernetes/cilium/network-policies.yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: backend-egress
  namespace: nxa-backend
spec:
  endpointSelector:
    matchLabels:
      app: nxa-backend
  egress:
    - toEndpoints:
        - matchLabels:
            app: nxa-db
      toPorts:
        - ports:
            - port: "5432"
              protocol: TCP
    - toEndpoints:
        - matchLabels:
            app: nxa-redis
      toPorts:
        - ports:
            - port: "6379"
              protocol: TCP
    - toEndpoints:
        - matchLabels:
            app: nxa-kafka
      toPorts:
        - ports:
            - port: "9093"
              protocol: TCP
    - toCIDR:
        - 0.0.0.0/0  # For external APIs (Lansweeper, Stripe, Cloud Providers)
      toPorts:
        - ports:
            - port: "443"
              protocol: TCP
        - ports:
            - port: "80"
              protocol: TCP
---
# Deny all ingress to tenant namespaces except from Traefik
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: tenant-ingress
  namespace: nxa-tenant-${TENANT_ID}
spec:
  endpointSelector: {}
  ingress:
    - fromEndpoints:
        - matchLabels:
            app: nxa-traefik-public
```

---

## 7. Observability Stack

### 7.1 Prometheus (Metrics)

```yaml
# platform/kubernetes/monitoring/prometheus.yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: nxa-prometheus
  namespace: monitoring
spec:
  replicas: 2
  retention: 30d
  retentionSize: 500GB
  storage:
    volumeClaimTemplate:
      spec:
        storageClassName: gp3
        resources:
          requests:
            storage: 500Gi
  resources:
    requests:
      memory: 8Gi
      cpu: "4"
  ruleSelector:
    matchLabels:
      team: platform
  serviceMonitorSelector:
    matchLabels:
      team: platform
  podMonitorSelector:
    matchLabels:
      team: platform
  remoteWrite:
    - url: https://cortex.nxa-internal:9009/api/v1/write
      tlsConfig:
        caFile: /etc/prometheus/certs/ca.crt
```

**Key Metrics (RED + USE + Business)**:
```prometheus
# RED Metrics (per service, per endpoint)
http_requests_total{service="nxa-backend",endpoint="/api/v1/assets",method="GET",status="200"}
http_request_duration_seconds_bucket{service="nxa-backend",endpoint="/api/v1/assets",le="0.5"}
http_requests_total{service="nxa-backend",status=~"5.."}

# USE Metrics (per resource)
container_cpu_usage_seconds_total{pod=~"nxa-backend-.*"}
container_memory_usage_bytes{pod=~"nxa-backend-.*"}
pgbouncer_pool_usage{pool="nexusasset"}

# Business Metrics
nxa_assets_total{tenant_id="tn_abc123",state="ACTIVE"}
nxa_sync_job_duration_seconds{site_id="site_001",status="success"}
nxa_invoice_total_amount{tenant_id="tn_abc123",currency="EUR"}
nxa_compliance_score{tenant_id="tn_abc123",framework="ISO_27001"}
```

### 7.2 Grafana (Dashboards)

```yaml
# platform/kubernetes/monitoring/grafana-dashboards.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboards
  namespace: monitoring
data:
  nxa-overview.json: |
    {
      "title": "NexusAsset Cloud - Platform Overview",
      "panels": [
        {"title": "Request Rate (RPS)", "type": "graph", "targets": [{"expr": "rate(http_requests_total[5m])"}]},
        {"title": "Error Rate", "type": "graph", "targets": [{"expr": "rate(http_requests_total{status=~\"5..\"}[5m])"}]},
        {"title": "Latency p99", "type": "graph", "targets": [{"expr": "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))"}]},
        {"title": "Active Tenants", "type": "stat", "targets": [{"expr": "count(nxa_tenant_status{status=\"ACTIVE\"})"}]},
        {"title": "Sync Throughput", "type": "graph", "targets": [{"expr": "rate(nxa_assets_processed_total[5m])"}]}
      ]
    }
  nxa-tenant-detail.json: |
    {
      "title": "NexusAsset Cloud - Tenant Detail",
      "panels": [
        {"title": "Asset Count by State", "type": "piechart", "targets": [{"expr": "nxa_assets_total{tenant_id=\"$tenant\"}"}]},
        {"title": "API Usage vs Quota", "type": "gauge", "targets": [{"expr": "nxa_api_calls_per_min{tenant_id=\"$tenant\"} / nxa_quota_api_calls_per_min{tenant_id=\"$tenant\"}"}]},
        {"title": "Compliance Posture", "type": "stat", "targets": [{"expr": "nxa_compliance_score{tenant_id=\"$tenant\"}"}]}
      ]
    }
```

### 7.3 Loki (Logs)

```yaml
# platform/kubernetes/monitoring/loki.yaml
apiVersion: loki.grafana.com/v1
kind: LokiStack
metadata:
  name: nxa-loki
  namespace: monitoring
spec:
  size: 1x.large
  storage:
    secretName: loki-s3-credentials
    type: s3
  tenants:
    mode: openshift-logging
  limits:
    global:
      ingestion:
        ingestionBurstSize: 10
        ingestionRate: 5
      queries:
        maxQueryParallelism: 32
```

**Log Format (JSON)**:
```json
{
  "timestamp": "2026-11-09T10:30:45.123456Z",
  "level": "INFO",
  "logger": "nexusasset.assets.use_cases.authorize_asset",
  "message": "Asset authorized",
  "trace_id": "0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6",
  "span_id": "a1b2c3d4e5f6",
  "tenant_id": "tn_abc123",
  "user_id": "usr_xyz789",
  "context": {
    "asset_id": "ast_123456",
    "from_state": "DISCOVERED",
    "to_state": "AUTHORIZED",
    "reason": "Policy evaluation passed"
  }
}
```

### 7.4 Tempo (Traces)

```yaml
# platform/kubernetes/monitoring/tempo.yaml
apiVersion: tempo.grafana.com/v1alpha1
kind: TempoStack
metadata:
  name: nxa-tempo
  namespace: monitoring
spec:
  storage:
    secretName: tempo-s3-credentials
    type: s3
  resources:
    total:
      limits:
        memory: 8Gi
        cpu: "4"
  template:
    queryFrontend:
      replicas: 2
    distributor:
      replicas: 3
    ingester:
      replicas: 3
    compactor:
      replicas: 1
```

**OpenTelemetry Instrumentation**:
```python
# backend/nexusasset/shared/kernel/infrastructure/observability/tracing.py
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.instrumentation.django import DjangoInstrumentor
from opentelemetry.instrumentation.redis import RedisInstrumentor
from opentelemetry.instrumentation.psycopg2 import Psycopg2Instrumentor
from opentelemetry.instrumentation.kafka import KafkaInstrumentor

def setup_tracing(service_name: str):
    provider = TracerProvider()
    provider.add_span_processor(
        BatchSpanProcessor(OTLPSpanExporter(endpoint="tempo.monitoring:4317"))
    )
    trace.set_tracer_provider(provider)
    
    # Auto-instrumentation
    DjangoInstrumentor().instrument()
    RedisInstrumentor().instrument()
    Psycopg2Instrumentor().instrument()
    KafkaInstrumentor().instrument()
    
    return trace.get_tracer(__name__)
```

### 7.5 Alerting (Alertmanager + PagerDuty)

```yaml
# platform/kubernetes/monitoring/alerts.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: nxa-critical-alerts
  namespace: monitoring
spec:
  groups:
    - name: nxa.platform.critical
      interval: 30s
      rules:
        - alert: ServiceDown
          expr: up{job=~"nxa-.*"} == 0
          for: 1m
          labels:
            severity: critical
            team: platform
          annotations:
            summary: "Service {{ $labels.job }} is down"
            runbook: "https://runbooks.nexusasset.cloud/service-down"
        
        - alert: HighErrorRate
          expr: |
            sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
            /
            sum(rate(http_requests_total[5m])) by (service)
            > 0.05
          for: 2m
          labels:
            severity: critical
            team: backend
          annotations:
            summary: "High error rate on {{ $labels.service }}"
            runbook: "https://runbooks.nexusasset.cloud/high-error-rate"
        
        - alert: DatabaseConnectionsHigh
          expr: pgbouncer_pool_usage > 0.85
          for: 5m
          labels:
            severity: warning
            team: database
          annotations:
            summary: "PgBouncer pool usage > 85%"
            runbook: "https://runbooks.nexusasset.cloud/db-connections-high"
        
        - alert: SyncJobFailure
          expr: increase(nxa_sync_job_status{status="failed"}[15m]) > 0
          for: 0m
          labels:
            severity: critical
            team: integration
          annotations:
            summary: "Lansweeper sync job failed for {{ $labels.site_id }}"
            runbook: "https://runbooks.nexusasset.cloud/sync-job-failure"
        
        - alert: ComplianceFindingCritical
          expr: nxa_compliance_finding_risk_score > 9
          for: 0m
          labels:
            severity: critical
            team: security
          annotations:
            summary: "Critical compliance finding: {{ $labels.finding_id }}"
            runbook: "https://runbooks.nexusasset.cloud/critical-finding"
```

---

## 8. Secrets & Configuration Management

### 8.1 Configuration Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              CONFIGURATION HIERARCHY                                                │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  1. DEFAULTS (Code)                                                                                 │
│     └─ backend/nexusasset/shared/kernel/config/defaults.py                                         │
│        • Base settings, feature flag defaults, timeout defaults                                    │
│                                                                                                     │
│  2. ENVIRONMENT (K8s ConfigMap)                                                                    │
│     └─ platform/kubernetes/configmaps/nxa-config.yaml                                              │
│        • Non-secret config per environment (dev/staging/prod)                                      │
│        • Feature flags, rate limits, retry policies                                                │
│                                                                                                     │
│  3. SECRETS (Vault → ExternalSecrets → K8s Secret)                                                 │
│     └─ Vault paths (see Section 6.1) → ExternalSecret CRDs → K8s Secrets                           │
│        • Database credentials, API keys, certificates, tokens                                      │
│                                                                                                     │
│  4. TENANT SETTINGS (Database)                                                                     │
│     └─ nxa_tenants.domain.TenantSettings (VO-TN-003)                                               │
│        • Per-tenant: custom fields, workflows, retention, integrations, quotas                     │
│                                                                                                     │
│  5. RUNTIME OVERRIDES (Feature Flags)                                                              │
│     └─ nxa_core.domain.FeatureFlag (AGG-CR-001) evaluated per request                              │
│        • Boolean/multivariate flags, targeting rules, gradual rollout                              │
│                                                                                                     │
│  RESOLUTION ORDER: 5 → 4 → 3 → 2 → 1 (highest wins)                                                │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Backup, Disaster Recovery & Business Continuity

### 9.1 Backup Strategy

| Component | Backup Method | Frequency | Retention | RPO | RTO | Verification |
|-----------|---------------|-----------|-----------|-----|-----|--------------|
| **PostgreSQL (RDS)** | Automated snapshots + PITR | Continuous (WAL) + Daily snapshot | 35 days | < 1 sec | < 15 min | Monthly restore test |
| **Redis (ElastiCache)** | Automated snapshots | Daily | 7 days | < 1 hr | < 30 min | Quarterly restore test |
| **Kafka (MSK)** | Topic replication (3x) + MirrorMaker to DR region | Continuous | 7 days hot | < 1 min | < 1 hr | Monthly failover test |
| **MinIO/S3** | Cross-region replication (CRR) | Continuous | 7 years (Glacier) | 0 | < 1 hr | Quarterly restore test |
| **Kubernetes (etcd)** | Velero + Restic to S3 | Daily | 30 days | < 1 hr | < 2 hr | Monthly cluster restore |
| **Vault** | Raft snapshot + S3 | Daily | 90 days | < 1 hr | < 30 min | Monthly unseal test |
| **Application Config** | Git (ArgoCD) | Continuous | Forever | 0 | < 5 min | Every deploy |

### 9.2 Disaster Recovery Topology

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              DISASTER RECOVERY TOPOLOGY                                             │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  PRIMARY REGION: eu-west-1                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  Active: All services, Primary DB, Kafka, Redis, MinIO                                      │   │
│  │  RPO: < 1 sec (sync DB replica), < 1 min (async Kafka mirror)                              │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                    │                              │                                              │
│         ┌──────────┴──────────┐        ┌──────────┴──────────┐                                  │
│         ▼                     ▼        ▼                     ▼                                  │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐                                  │
│  │ SYNC REPLICA  │    │ ASYNC REPLICA │    │ ASYNC REPLICA │                                  │
│  │ eu-central-1  │    │ us-east-1     │    │ ap-southeast-1│                                  │
│  │ (EU Residency)│    │ (DR)          │    │ (APAC DR)     │                                  │
│  ├───────────────┤    ├───────────────┤    ├───────────────┤                                  │
│  │ RPO = 0       │    │ RPO < 1 hr    │    │ RPO < 1 hr    │                                  │
│  │ RTO < 30 min  │    │ RTO < 4 hr    │    │ RTO < 4 hr    │                                  │
│  │ DB: Sync      │    │ DB: Async     │    │ DB: Async     │                                  │
│  │ Kafka: Mirror │    │ Kafka: Mirror │    │ Kafka: Mirror │                                  │
│  │ MinIO: CRR    │    │ MinIO: CRR    │    │ MinIO: CRR    │                                  │
│  │ K8s: Standby  │    │ K8s: Standby  │    │ K8s: Standby  │                                  │
│  └───────────────┘    └───────────────┘    └───────────────┘                                  │
│                                                                                                     │
│  FAILOVER PROCESS:                                                                                │
│  1. Health check fails (Prometheus alert)                                                         │
│  2. Incident declared (PagerDuty)                                                                   │
│  3. Runbook executed: DNS failover (Cloudflare), DB promotion, K8s scale-up                       │
│  4. Validation: Smoke tests, data integrity checks                                                │
│  5. Traffic cutover complete                                                                        │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 9.3 Chaos Engineering

```yaml
# platform/chaos/experiments.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: backend-pod-kill
  namespace: chaos-testing
spec:
  action: pod-kill
  mode: one
  selector:
    namespaces:
      - nxa-backend
    labelSelectors:
      app: nxa-backend
  scheduler:
    cron: "@every 1h"
---
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: db-latency
  namespace: chaos-testing
spec:
  action: delay
  mode: all
  selector:
    namespaces:
      - nxa-backend
    labelSelectors:
      app: nxa-backend
  delay:
    latency: "100ms"
    correlation: "25"
    jitter: "50ms"
  scheduler:
    cron: "@every 2h"
---
apiVersion: chaos-mesh.org/v1alpha1
kind: TimeChaos
metadata:
  name: clock-skew
  namespace: chaos-testing
spec:
  action: time
  mode: one
  selector:
    namespaces:
      - nxa-backend
    labelSelectors:
      app: nxa-backend
  timeOffset: "-5m"
  scheduler:
    cron: "@every 4h"
```

---

## 10. Capacity Planning & Autoscaling

### 10.1 Resource Quotas (Per Tenant)

```yaml
# platform/kubernetes/quotas/tenant-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-quota
  namespace: nxa-tenant-{tenant_id}
spec:
  hard:
    cpu: "4"
    memory: 8Gi
    pods: "20"
    services: "10"
    persistentvolumeclaims: "5"
    requests.storage: 50Gi
    limits.cpu: "8"
    limits.memory: 16Gi
---
# LimitRange for default container resources
apiVersion: v1
kind: LimitRange
metadata:
  name: container-limits
  namespace: nxa-tenant-{tenant_id}
spec:
  limits:
    - type: Container
      default:
        cpu: "500m"
        memory: "1Gi"
      defaultRequest:
        cpu: "100m"
        memory: "256Mi"
      max:
        cpu: "2"
        memory: "4Gi"
      min:
        cpu: "50m"
        memory: "64Mi"
```

### 10.2 Horizontal Pod Autoscaler (HPA)

```yaml
# platform/kubernetes/hpa/backend-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nxa-backend-hpa
  namespace: nxa-backend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nxa-backend
  minReplicas: 3
  maxReplicas: 50
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "100"
    - type: External
      external:
        metric:
          name: kafka_consumer_group_lag
          selector:
            matchLabels:
              consumergroup: nxa-backend
        target:
          type: Value
          value: "1000"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 50
          periodSeconds: 60
        - type: Pods
          value: 4
          periodSeconds: 60
      selectPolicy: Max
```

### 10.3 KEDA for Worker Scaling

```yaml
# platform/kubernetes/keda/worker-scaledobject.yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: nxa-worker-scaled
  namespace: nxa-worker
spec:
  scaleTargetRef:
    name: nxa-worker
  pollingInterval: 15
  cooldownPeriod: 300
  minReplicaCount: 3
  maxReplicaCount: 30
  triggers:
    - type: kafka
      metadata:
        bootstrapServers: nxa-kafka.kafka:9093
        consumerGroup: nxa-worker-keda
        topic: nxa.lansweeper.sync
        lagThreshold: "100"
    - type: prometheus
      metadata:
        serverAddress: http://prometheus.monitoring:9090
        metricName: celery_queue_depth
        query: celery_queue_depth{queue="lansweeper_sync"}
        threshold: "50"
    - type: cpu
      metadata:
        type: Utilization
        value: "70"
```

---

## 11. Infrastructure as Code (Terraform/Ansible)

### 11.1 Terraform Structure

```
platform/terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       └── terraform.tfvars
├── modules/
│   ├── vpc/
│   ├── eks/
│   ├── rds/
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

### 11.2 Ansible Playbooks (Configuration Management)

```yaml
# platform/ansible/playbooks/configure-nodes.yml
- name: Configure Kubernetes Nodes
  hosts: k8s_nodes
  become: yes
  roles:
    - role: common
      tags: [always]
    - role: docker
      tags: [container-runtime]
    - role: kubelet
      tags: [kubernetes]
    - role: cilium
      tags: [cni]
    - role: monitoring-agent
      tags: [observability]
    - role: security-agent
      tags: [security]
    - role: log-forwarder
      tags: [logging]
```

---

## 12. Environment Strategy

| Environment | Purpose | Clusters | Data | Deploy Trigger | Access |
|-------------|---------|----------|------|----------------|--------|
| **Local** | Developer laptop | kind (1 node) | SQLite / Testcontainers | Manual | Developer |
| **Dev** | Integration testing | 1 cluster (2 AZ) | Shared DB (schema per dev) | Push to `dev` branch | Team |
| **Staging** | Pre-production validation | 1 cluster (3 AZ) | Production-like (anonymized) | PR merge to `main` | Team + QA |
| **Production** | Live traffic | 3 regions (3 AZ each) | Real tenant data | Git tag + ArgoCD promotion | Platform + On-call |

---

## 13. Diagrams

| Diagram ID | Title | Type | Location | Description |
|------------|-------|------|----------|-------------|
| **DIA-057** | Platform Topology | Deployment | Section 1.1 | Multi-region, multi-AZ with edge |
| **DIA-058** | Kubernetes Namespace Architecture | Component | Section 2.2 | Platform, tenant, shared namespaces |
| **DIA-059** | Traefik Edge Routing | Sequence | Section 3.1 | Request → Cloudflare → Traefik → Service |
| **DIA-060** | mTLS with SPIFFE/Cilium | Component | Section 3.2 | Workload identity, policy enforcement |
| **DIA-061** | PostgreSQL RLS Architecture | Component | Section 4.2 | Tenant isolation at DB level |
| **DIA-062** | Redis Cluster Topology | Deployment | Section 4.4 | Shards, replicas, sentinel |
| **DIA-063** | Kafka Topic Design | Component | Section 5.2 | Topics, partitions, retention |
| **DIA-064** | Vault Secrets Flow | Sequence | Section 6.1 | App → Vault → K8s Secret |
| **DIA-065** | Network Policies (Cilium) | Component | Section 6.3 | Ingress/Egress rules |
| **DIA-066** | Observability Stack | Component | Section 7 | Prometheus, Grafana, Loki, Tempo, OTel |
| **DIA-067** | DR Topology | Deployment | Section 9.2 | Primary, sync/async replicas |
| **DIA-068** | Autoscaling Architecture | Component | Section 10 | HPA, KEDA, VPA, Cluster Autoscaler |

---

## 14. Tables

| Table ID | Title | Location | Row Count |
|----------|-------|----------|-----------|
| **TBL-131** | Service Inventory | Section 1.2 | 10 |
| **TBL-132** | Cluster Specification | Section 2.1 | 1 |
| **TBL-133** | Namespace Architecture | Section 2.2 | 3 |
| **TBL-134** | Workload Deployment Patterns | Section 2.3 | 4 |
| **TBL-135** | Domain Strategy | Section 3.3 | 6 |
| **TBL-136** | RDS Configuration | Section 4.1 | 1 |
| **TBL-137** | RLS Policies | Section 4.2 | 1 |
| **TBL-138** | PgBouncer Config | Section 4.3 | 1 |
| **TBL-139** | Redis Usage Patterns | Section 4.4 | 7 |
| **TBL-140** | MinIO Buckets | Section 4.5 | 5 |
| **TBL-141** | Kafka Topic Design | Section 5.2 | 11 |
| **TBL-142** | Vault Secret Paths | Section 6.1 | 10 |
| **TBL-143** | Key Metrics (RED/USE/Business) | Section 7.1 | 10 |
| **TBL-144** | Critical Alerts | Section 7.5 | 5 |
| **TBL-145** | Configuration Hierarchy | Section 8.1 | 5 |
| **TBL-146** | Backup Strategy | Section 9.1 | 7 |
| **TBL-147** | DR Replica Specs | Section 9.2 | 3 |
| **TBL-148** | Tenant Resource Quotas | Section 10.1 | 1 |
| **TBL-149** | HPA Metrics | Section 10.2 | 3 |
| **TBL-150** | KEDA Triggers | Section 10.3 | 3 |
| **TBL-151** | Environment Strategy | Section 12 | 5 |
| **TBL-152** | Diagram Index | Section 13 | 12 |
| **TBL-153** | Table Index | Section 14 | 23 |

---

---

**End of DOC-06 — NexusAsset Cloud Infrastructure Architecture**

*Document Control: Version 1.0.0 | Classification: Internal — Confidential | Next Review: 2026-12-09*

*For questions or updates: docs@nexusasset.cloud | Architecture Review Board: arch-review@nexusasset.cloud*