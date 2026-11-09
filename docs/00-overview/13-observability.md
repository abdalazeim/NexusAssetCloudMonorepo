# DOC-13: Observability Architecture

---

| **Document ID** | DOC-13 |
|-----------------|--------|
| **Title** | Observability Architecture |
| **Version** | 1.0.0 |
| **Status** | ✓ Approved |
| **Owner** | Platform Engineering |
| **Last Updated** | 2026-11-09 |
| **Classification** | Internal — Confidential |
| **Related Docs** | [DOC-05 Application Architecture](../00-overview/05-application-architecture.md), [DOC-06 Infrastructure Architecture](../00-overview/06-infrastructure-architecture.md), [DOC-07 API Design](../00-overview/07-api-design.md), [DOC-09 Frontend Architecture](../00-overview/09-frontend-architecture.md), [DOC-10 Testing Strategy](../00-overview/10-testing-strategy.md), [DOC-11 DevOps & CI/CD](../00-overview/11-devops-cicd.md), [DOC-12 Security Architecture](../00-overview/12-security-architecture.md) |
| **Traceability** | REQ-NFR-OBS-###, SLO/RB-###, ADR-019..024 |

---

## 1. Executive Summary

NexusAsset Cloud implements a **full-stack observability platform** built on the **OpenTelemetry-native** stack: **Prometheus** (metrics), **Grafana** (visualization), **Loki** (logs), **Tempo** (traces), and **OpenTelemetry Collector** (telemetry pipeline). This document defines the architecture, SLO/SLI framework, alerting strategy, dashboard taxonomy, and operational procedures ensuring **99.99% availability** with **<5 min MTTR** for P1 incidents.

> **Note:** All observability components run in the dedicated `observability` namespace with dedicated node pools, encrypted storage, and network policies restricting access to authorized personnel only.

---

## 2. Observability Stack Architecture

### 2.1 High-Level Topology

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        NEXUSASSET CLOUD OBSERVABILITY PLANE                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │  APP TIER    │    │  BFF TIER    │    │  INFRA TIER  │    │  FRONTEND    │  │
│  │  (Django)    │    │  (Next.js)   │    │  (K8s/DB)    │    │  (Browser)   │  │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘    └──────┬───────┘  │
│         │                   │                   │                   │           │
│         ▼                   ▼                   ▼                   ▼           │
│  ┌──────────────────────────────────────────────────────────────────────────┐   │
│  │                    OPENTELEMETRY COLLECTOR (DAEMONSET)                   │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐    │   │
│  │  │Receivers│→ │Processors│→ │Exporters │  (OTLP/HTTP/gRPC)             │    │   │
│  │  │OTLP,    │  │Batch,   │  │Prom,    │                                   │    │   │
│  │  │Prom,    │  │Memory   │  │Tempo,   │                                   │    │   │
│  │  │Filelog  │  │Limiter, │  │Loki,    │                                   │    │   │
│  │  │         │  │Transform│  │OTLP     │                                   │    │   │
│  │  └─────────┘  └─────────┘  └─────────┘                                   │    │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│         │                   │                   │                   │           │
│         ▼                   ▼                   ▼                   ▼           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │  PROMETHEUS  │  │    TEMPO     │  │     LOKI     │  │   GRAFANA    │       │
│  │  (Metrics)   │  │  (Traces)    │  │   (Logs)     │  │  (Dashboards)│       │
│  │              │  │              │  │              │  │              │       │
│  │  • 15d hot   │  │  • 7d hot    │  │  • 30d hot   │  │  • Provisioned│      │
│  │  • 13m cold  │  │  • 13m cold  │  │  • 13m cold  │  │  • GitOps     │      │
│  │  • HA pair   │  │  • HA pair   │  │  • HA pair   │  │  • RBAC       │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘       │
│         │                 │                 │                 │                │
│         └─────────────────┴─────────────────┴─────────────────┘                │
│                                   │                                            │
│                                   ▼                                            │
│                    ┌──────────────────────────────┐                           │
│                    │      ALERTMANAGER HA         │                           │
│                    │  • PagerDuty / Slack / Email │                           │
│                    │  • Routing / Inhibition      │                           │
│                    │  • Silences / Webhooks       │                           │
│                    └──────────────────────────────┘                           │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Component Specifications

| Component | Version | Deployment | Replicas | Storage | Retention (Hot/Cold) | HA Mode |
|-----------|---------|------------|----------|---------|---------------------|---------|
| **OpenTelemetry Collector** | 0.118+ | DaemonSet + Deployment | 3+ (per AZ) | N/A | N/A | Active-Active |
| **Prometheus** | 2.53+ | StatefulSet (Thanos sidecar) | 2 per region | 500 GiB SSD | 15d / 13mo (Thanos) | HA Pair + Thanos |
| **Tempo** | 2.4+ | Distributed (monolithic for dev) | 3+ | 1 TiB S3/MinIO | 7d / 13mo | HA + Compactor |
| **Loki** | 2.9+ | Distributed (read/write) | 3+ read, 3+ write | 2 TiB S3/MinIO | 30d / 13mo | HA + Ruler |
| **Grafana** | 11.1+ | Deployment | 3+ | 10 GiB PVC (SQLite/Postgres) | N/A | Active-Active |
| **Alertmanager** | 0.27+ | StatefulSet | 3 | 10 GiB PVC | N/A | HA Cluster |
| **Thanos** | 0.35+ | Sidecar + Query + Compactor | 2+ | S3/MinIO | 13mo | HA |
| **Mimir** (future) | — | — | — | — | — | — |

> **Tip:** Production uses **Thanos** for long-term metric storage and global query. Development uses standalone Prometheus with 30d retention.

---

## 3. Telemetry Data Model

### 3.1 Metric Naming Convention (Prometheus)

```
<domain>_<subsystem>_<metric_name>_<unit>
```

| Domain | Subsystem | Examples |
|--------|-----------|----------|
| `nxa` | `api` | `nxa_api_requests_total`, `nxa_api_request_duration_seconds` |
| `nxa` | `worker` | `nxa_worker_jobs_total`, `nxa_worker_job_duration_seconds` |
| `nxa` | `db` | `nxa_db_connections_active`, `nxa_db_query_duration_seconds` |
| `nxa` | `cache` | `nxa_cache_hits_total`, `nxa_cache_misses_total` |
| `nxa` | `sync` | `nxa_sync_assets_processed_total`, `nxa_sync_duration_seconds` |
| `infra` | `k8s` | `infra_k8s_pod_restarts_total`, `infra_k8s_node_cpu_usage_ratio` |
| `infra` | `net` | `infra_net_bytes_sent_total`, `infra_net_errors_total` |

### 3.2 Standard Labels (Mandatory)

| Label | Description | Cardinality |
|-------|-------------|-------------|
| `tenant_id` | Tenant identifier (hashed for PII) | High (10k+) |
| `environment` | `prod` \| `staging` \| `dev` | Low (3) |
| `region` | AWS region / AZ | Low (3-6) |
| `service` | `nxa-api`, `nxa-bff`, `nxa-worker`, `nxa-sync` | Medium (10) |
| `version` | Semantic version (e.g., `1.2.3`) | Medium (50) |
| `route` | Normalized API route (`/api/v1/assets`) | Medium (200) |
| `method` | HTTP method | Low (6) |
| `status_code` | HTTP status | Low (20) |
| `le` | Histogram bucket (auto) | Low (10) |

> **Warning:** Never add unbounded labels (e.g., `user_id`, `request_id`, `ip_address`). Use `tenant_id` hash instead.

### 3.3 Log Structure (Structured JSON)

```json
{
  "timestamp": "2026-11-09T12:34:56.789Z",
  "level": "info",
  "logger": "nxa.assets.services.sync",
  "message": "Asset synchronization completed",
  "trace_id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "span_id": "a1b2c3d4e5f6a1b2",
  "tenant_id": "tnt_abc123",
  "service": "nxa-sync-worker",
  "version": "1.4.2",
  "environment": "prod",
  "region": "us-east-1",
  "fields": {
    "sync_id": "sync_xyz789",
    "assets_processed": 15420,
    "assets_created": 123,
    "assets_updated": 456,
    "assets_deleted": 7,
    "duration_ms": 34210
  }
}
```

### 3.4 Trace Context Propagation

| Header | Format | Required |
|--------|--------|----------|
| `traceparent` | `00-<trace-id>-<span-id>-01` (W3C) | Yes |
| `tracestate` | Vendor-specific (e.g., `nxa=tenant:tnt_abc123`) | Yes |
| `x-b3-traceid` | B3 single-header (legacy) | Optional |
| `x-b3-spanid` | B3 single-header (legacy) | Optional |

---

## 4. SLO / SLI Framework

### 4.1 Service Level Objectives (SLOs)

| SLO ID | Service | SLI | Target | Window | Error Budget | Burn Rate Alert |
|--------|---------|-----|--------|--------|--------------|-----------------|
| **SLO-RB-001** | API Gateway | Availability (HTTP 2xx/3xx / total) | 99.99% | 28d | 5.04 min/mo | 2% (1h), 10% (6h) |
| **SLO-RB-002** | API Gateway | Latency P99 < 500ms | 99.9% | 28d | 43.2 min/mo | 2% (1h), 10% (6h) |
| **SLO-RB-003** | Asset API | Availability | 99.95% | 28d | 20.16 min/mo | 2% (1h), 10% (6h) |
| **SLO-RB-004** | Asset API | Latency P95 < 300ms | 99.5% | 28d | 3.6 h/mo | 5% (1h), 20% (6h) |
| **SLO-RB-005** | Sync Workers | Job Success Rate | 99.9% | 28d | 4.32 min/mo | 2% (1h), 10% (6h) |
| **SLO-RB-006** | Sync Workers | End-to-End Latency P95 < 15min | 99% | 28d | 7.2 h/mo | 5% (1h), 20% (6h) |
| **SLO-RB-007** | BFF / Frontend | Availability | 99.99% | 28d | 5.04 min/mo | 2% (1h), 10% (6h) |
| **SLO-RB-008** | BFF / Frontend | FCP < 1.8s (RUM) | 95% | 28d | 36 h/mo | 10% (1h), 30% (6h) |
| **SLO-RB-009** | BFF / Frontend | TTI < 3.5s (RUM) | 95% | 28d | 36 h/mo | 10% (1h), 30% (6h) |
| **SLO-RB-010** | Database | Availability (write) | 99.99% | 28d | 5.04 min/mo | 2% (1h), 10% (6h) |
| **SLO-RB-011** | Database | Replication Lag P99 < 1s | 99.9% | 28d | 43.2 min/mo | 2% (1h), 10% (6h) |
| **SLO-RB-012** | Cache | Hit Ratio > 95% | 99.9% | 28d | 43.2 min/mo | 2% (1h), 10% (6h) |

### 4.2 Error Budget Policy

| Burn Rate | Window | Action | Escalation |
|-----------|--------|--------|------------|
| **2%** | 1h | **Page** (P1) | On-call + Slack #incidents |
| **5%** | 1h | **Page** (P2) | On-call + Slack #incidents |
| **10%** | 6h | **Alert** (P2) | On-call (next business hour) |
| **20%** | 6h | **Alert** (P3) | Team lead notification |
| **50%** | 1d | **Review** | Sprint retrospective |

> **Note:** Error budget calculated as `(1 - SLO_target) * window_duration`. Multi-window, multi-burn-rate alerting per [Google SRE Workbook](https://sre.google/workbook/alerting/).

### 4.3 SLI Definitions (PromQL)

```promql
# SLO-RB-001: API Availability
sum(rate(nxa_api_requests_total{status_code=~"2..|3.."}[5m]))
/
sum(rate(nxa_api_requests_total[5m]))

# SLO-RB-002: API Latency P99 < 500ms
histogram_quantile(0.99,
  sum(rate(nxa_api_request_duration_seconds_bucket[5m])) by (le, route)
) < 0.5

# SLO-RB-005: Sync Job Success Rate
sum(rate(nxa_sync_jobs_total{status="success"}[5m]))
/
sum(rate(nxa_sync_jobs_total[5m]))

# SLO-RB-010: DB Write Availability
sum(rate(infra_db_write_requests_total{status="success"}[5m]))
/
sum(rate(infra_db_write_requests_total[5m]))
```

---

## 5. Alerting Architecture

### 5.1 Alert Classification

| Severity | Prefix | Response Time | Escalation | Examples |
|----------|--------|---------------|------------|----------|
| **Critical (P1)** | `critical_` | **5 min** | Page → On-call → Manager (15m) → Director (30m) | SLO burn 2%/1h, DB down, K8s control plane |
| **High (P2)** | `high_` | **15 min** | Page → On-call → Team Lead (30m) | SLO burn 5%/1h, Sync lag > 30m, Cache miss spike |
| **Warning (P3)** | `warning_` | **1 hour** | Slack → On-call (next business hour) | Disk > 80%, CPU > 85%, Replica lag > 5s |
| **Info (P4)** | `info_` | **24 hours** | Slack (batched) | New deployment, Config change, Cert expiry > 30d |

### 5.2 Alert Routing (Alertmanager)

```yaml
# alertmanager-config.yaml (GitOps managed)
route:
  group_by: ['alertname', 'tenant_id', 'service', 'region']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'default'
  routes:
  - match:
      severity: critical
    receiver: 'pagerduty-critical'
    continue: true
  - match:
      severity: high
    receiver: 'pagerduty-high'
    continue: true
  - match:
      severity: warning
    receiver: 'slack-warning'
  - match:
      severity: info
    receiver: 'slack-info'
    group_interval: 1h

receivers:
- name: 'default'
  email_configs:
  - to: 'platform-oncall@nexusasset.com'
- name: 'pagerduty-critical'
  pagerduty_configs:
  - service_key: ${PAGERDUTY_CRITICAL_KEY}
    severity: critical
- name: 'pagerduty-high'
  pagerduty_configs:
  - service_key: ${PAGERDUTY_HIGH_KEY}
    severity: error
- name: 'slack-warning'
  slack_configs:
  - api_url: ${SLACK_WEBHOOK}
    channel: '#alerts-warning'
    title: '⚠️ {{ .GroupLabels.alertname }}'
- name: 'slack-info'
  slack_configs:
  - api_url: ${SLACK_WEBHOOK}
    channel: '#alerts-info'
    title: 'ℹ️ {{ .GroupLabels.alertname }}'

inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  equal: ['alertname', 'tenant_id', 'service', 'region']
```

### 5.3 Core Alert Rules (PrometheusRule)

```yaml
# nxa-critical-alerts.yaml
groups:
- name: nxa.slo.burn.rate
  interval: 30s
  rules:
  - alert: critical_api_availability_burn_rate_1h
    expr: |
      (
        1 - (
          sum(rate(nxa_api_requests_total{status_code=~"2..|3.."}[1h]))
          /
          sum(rate(nxa_api_requests_total[1h]))
        )
      ) / (1 - 0.9999) > 2
    for: 2m
    labels:
      severity: critical
      slo: SLO-RB-001
      team: platform
    annotations:
      summary: "API Availability SLO burn rate > 2% (1h window)"
      description: "Error budget consuming at {{ $value | humanizePercentage }}x rate. Remaining budget: {{ $value | humanizeDuration }}"
      runbook_url: "https://runbooks.nexusasset.com/slo-api-availability"

  - alert: critical_api_latency_p99_burn_rate_1h
    expr: |
      (
        histogram_quantile(0.99, sum(rate(nxa_api_request_duration_seconds_bucket[1h])) by (le, route)) > 0.5
      ) / (1 - 0.999) > 2
    for: 2m
    labels:
      severity: critical
      slo: SLO-RB-002
      team: backend
    annotations:
      summary: "API Latency P99 SLO burn rate > 2% (1h window)"
      runbook_url: "https://runbooks.nexusasset.com/slo-api-latency"

- name: nxa.infrastructure
  interval: 30s
  rules:
  - alert: critical_kubernetes_control_plane_down
    expr: up{job="kubernetes-apiserver"} == 0
    for: 1m
    labels:
      severity: critical
      team: platform
    annotations:
      summary: "Kubernetes API server unreachable"
      runbook_url: "https://runbooks.nexusasset.com/k8s-control-plane"

  - alert: critical_database_primary_down
    expr: pg_up{role="primary"} == 0
    for: 30s
    labels:
      severity: critical
      team: platform
    annotations:
      summary: "PostgreSQL primary unavailable"
      runbook_url: "https://runbooks.nexusasset.com/db-primary-down"

  - alert: high_database_replication_lag
    expr: pg_replication_lag_seconds > 30
    for: 5m
    labels:
      severity: high
      team: platform
    annotations:
      summary: "DB replication lag > 30s"
      runbook_url: "https://runbooks.nexusasset.com/db-replication-lag"

- name: nxa.application.sync
  interval: 60s
  rules:
  - alert: high_sync_job_failure_rate
    expr: |
      sum(rate(nxa_sync_jobs_total{status="failed"}[15m]))
      /
      sum(rate(nxa_sync_jobs_total[15m])) > 0.05
    for: 5m
    labels:
      severity: high
      team: backend
      tenant_specific: "true"
    annotations:
      summary: "Sync job failure rate > 5% for tenant {{ $labels.tenant_id }}"
      runbook_url: "https://runbooks.nexusasset.com/sync-failures"

  - alert: high_sync_lag
    expr: time() - nxa_sync_last_success_timestamp_seconds > 1800
    for: 10m
    labels:
      severity: high
      team: backend
    annotations:
      summary: "No successful sync for {{ $labels.tenant_id }} in 30 min"
      runbook_url: "https://runbooks.nexusasset.com/sync-lag"
```

### 5.4 Alert Testing & Validation

| Test | Frequency | Tool | Success Criteria |
|------|-----------|------|------------------|
| **Alert Rule Syntax** | PR merge | `promtool check rules` | Zero errors |
| **Alert Firing** | Weekly | `amtool` + synthetic metrics | Fires within 2x `for` duration |
| **Notification Delivery** | Daily | Alertmanager test receiver | PagerDuty/Slack receive within 30s |
| **Inhibition Logic** | Monthly | Integration test suite | Critical inhibits warning for same labels |
| **Runbook Accuracy** | Quarterly | Fire drill | On-call resolves using runbook < 15 min |

---

## 6. Dashboard Taxonomy

### 6.1 Dashboard Categories

| Category | Prefix | Audience | Update Cadence | Examples |
|----------|--------|----------|----------------|----------|
| **Executive** | `exec-` | Leadership, PM | Monthly | Business KPIs, SLO compliance, Cost |
| **Service Overview** | `svc-` | On-call, SRE | Real-time | Golden signals, RED/USE, Error budget |
| **Component Deep-Dive** | `cmp-` | Engineers | Real-time | DB, Cache, Queue, Sync workers |
| **Tenant-Specific** | `tnt-` | CSM, Support | Real-time | Per-tenant health, Usage, Quotas |
| **Infrastructure** | `infra-` | Platform, SRE | Real-time | K8s, Network, Storage, Nodes |
| **Business** | `biz-` | Product, Sales | Hourly | Activation, Churn, Feature adoption |
| **Security** | `sec-` | SecOps | Real-time | Auth failures, WAF blocks, Vuln scans |

### 6.2 Golden Signals Dashboards (Per Service)

```json
{
  "dashboard": "svc-nxa-api-golden-signals",
  "panels": [
    {
      "title": "Traffic (RPS)",
      "type": "timeseries",
      "targets": [
        {"expr": "sum(rate(nxa_api_requests_total[1m])) by (route)", "legendFormat": "{{route}}"}
      ]
    },
    {
      "title": "Errors (Rate %)",
      "type": "timeseries",
      "targets": [
        {"expr": "sum(rate(nxa_api_requests_total{status_code=~\"5..\"}[1m])) by (route) / sum(rate(nxa_api_requests_total[1m])) by (route)", "legendFormat": "{{route}}"}
      ]
    },
    {
      "title": "Latency P50/P95/P99",
      "type": "timeseries",
      "targets": [
        {"expr": "histogram_quantile(0.50, sum(rate(nxa_api_request_duration_seconds_bucket[5m])) by (le, route))", "legendFormat": "P50 {{route}}"},
        {"expr": "histogram_quantile(0.95, sum(rate(nxa_api_request_duration_seconds_bucket[5m])) by (le, route))", "legendFormat": "P95 {{route}}"},
        {"expr": "histogram_quantile(0.99, sum(rate(nxa_api_request_duration_seconds_bucket[5m])) by (le, route))", "legendFormat": "P99 {{route}}"}
      ]
    },
    {
      "title": "Saturation (CPU/Mem/Connections)",
      "type": "timeseries",
      "targets": [
        {"expr": "sum(container_cpu_usage_seconds_total{pod=~\"nxa-api-.*\"}) by (pod) / sum(container_spec_cpu_quota/container_spec_cpu_period{pod=~\"nxa-api-.*\"}) by (pod)", "legendFormat": "CPU {{pod}}"},
        {"expr": "sum(container_memory_working_set_bytes{pod=~\"nxa-api-.*\"}) by (pod) / sum(container_spec_memory_limit_bytes{pod=~\"nxa-api-.*\"}) by (pod)", "legendFormat": "Mem {{pod}}"}
      ]
    },
    {
      "title": "Error Budget Remaining",
      "type": "gauge",
      "targets": [
        {"expr": "1 - (sum(rate(nxa_api_requests_total{status_code=~\"5..\"}[28d])) / sum(rate(nxa_api_requests_total[28d]))) / (1 - 0.9999)", "legendFormat": "Budget %"}
      ]
    }
  ]
}
```

### 6.3 RED Method Dashboards (Request/Error/Duration)

| Dashboard | Service | Key Panels |
|-----------|---------|------------|
| `svc-nxa-api-red` | API Gateway | RPS by route, Error rate by code, Latency heatmap |
| `svc-nxa-bff-red` | BFF | RPS by operation, GraphQL error rate, Resolver latency |
| `svc-nxa-sync-red` | Sync Workers | Jobs/min, Failure rate by source, Duration histogram |
| `svc-nxa-notifications-red` | Notifications | Events processed, Delivery failures, Channel latency |

### 6.4 USE Method Dashboards (Utilization/Saturation/Errors)

| Dashboard | Component | Key Panels |
|-----------|-----------|------------|
| `cmp-postgresql-use` | Primary/Replica | CPU/Mem/IO utilization, Connection saturation, Query errors |
| `cmp-redis-use` | Cluster/Standalone | Memory utilization, Key eviction rate, Command errors |
| `cmp-kafka-use` | Brokers | Disk/Network utilization, Partition lag, Controller elections |
| `cmp-kubernetes-use` | Nodes/Pods | CPU/Mem/Pod saturation, OOM kills, Restart rate |

### 6.5 Provisioning (GitOps)

All dashboards defined as **Grafonnet/Jsonnet** in `observability/dashboards/` and deployed via **Grafana Operator** or **ArgoCD ApplicationSet**.

```jsonnet
// dashboards/svc-nxa-api-golden-signals.libsonnet
local grafana = import 'grafonnet/grafana.libsonnet';
local dashboard = grafana.dashboard.new('svc-nxa-api-golden-signals')
  .uid('svc-nxa-api-golden-signals')
  .tags(['service', 'api', 'golden-signals', 'auto-provisioned'])
  .timezone('utc')
  .refresh('30s')
  .time('now-1h', 'now')
  .addVariable(grafana.variable.query('region', 'Region', 'label_values(nxa_api_requests_total, region)'))
  .addVariable(grafana.variable.query('tenant', 'Tenant', 'label_values(nxa_api_requests_total{region=~"$region"}, tenant_id)'))
  .addPanel(/* Traffic panel */)
  .addPanel(/* Errors panel */)
  .addPanel(/* Latency panel */)
  .addPanel(/* Saturation panel */)
  .addPanel(/* Error Budget panel */);
```

---

## 7. Distributed Tracing

### 7.1 Trace Architecture

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        DISTRIBUTED TRACE FLOW                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  CLIENT (Browser)                                                               │
│       │                                                                         │
│       │ traceparent: 00-a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4-a1b2c3d4e5f6a1b2-01   │
│       ▼                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                         TRAEFIK INGRESS (Span: ingress)                 │   │
│  │  • Adds: net.host.name, net.host.port, http.route, http.target          │   │
│  │  • Injects trace context to upstream                                    │   │
│  └────────────────────────────────┬────────────────────────────────────────┘   │
│                                   │                                             │
│               ┌───────────────────┼───────────────────┐                        │
│               ▼                   ▼                   ▼                        │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐             │
│  │   BFF (Next.js)  │  │  API (Django)    │  │  Sync Worker     │             │
│  │  Span: bff.render│  │  Span: api.get   │  │  Span: sync.run  │             │
│  │  • Server timing │  │  • DB queries    │  │  • Lancsweeper   │             │
│  │  • Cache ops     │  │  • Cache ops     │  │    GraphQL calls │             │
│  │  • RSC streams   │  │  • Auth checks   │  │  • DB writes     │             │
│  └────────┬─────────┘  └────────┬─────────┘  └────────┬─────────┘             │
│           │                     │                     │                        │
│           ▼                     ▼                     ▼                        │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                    OPENTELEMETRY COLLECTOR (Receiver)                   │   │
│  │  • Batch spans (512 spans or 5s)                                        │   │
│  │  • Add resource attributes (service, version, k8s.pod, tenant)          │   │
│  │  • Transform: scrub PII, normalize names                                │   │
│  │  • Export to Tempo (OTLP/gRPC)                                          │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                   │                                             │
│                                   ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                          TEMPO (Storage + Query)                        │   │
│  │  • Ingest: Distributors → Ingester → S3/MinIO (blocks)                 │   │
│  │  • Index: TraceID → Block mapping (Bloom filters)                      │   │
│  │  • Query: Querier → Query-frontend (parallel)                          │   │
│  │  • Retention: Compactor (merge blocks, TTL)                            │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Span Naming Convention

| Operation Type | Span Name Format | Example |
|----------------|------------------|---------|
| HTTP Server | `<method> <route>` | `GET /api/v1/assets` |
| HTTP Client | `<method> <host><path>` | `POST api.lansweeper.com/graphql` |
| Database | `<db.system> <operation> <table>` | `postgresql SELECT assets` |
| Cache | `<cache.system> <operation> <key_prefix>` | `redis GET asset:detail:` |
| Message Queue | `<messaging.system> <operation> <topic>` | `kafka PRODUCE nxa.asset.events` |
| Background Job | `job.<queue>.<task_name>` | `job.sync.full_asset_sync` |
| Custom Business | `<domain>.<action>` | `asset.reconcile`, `license.calculate` |

### 7.3 Standard Span Attributes (Semantic Conventions)

```python
# nxa_core/telemetry/span_attributes.py
from opentelemetry.semconv.trace import SpanAttributes

NXA_SPAN_ATTRIBUTES = {
    # Tenant context (always present)
    "nxa.tenant.id": "tenant_id_hash",           # Hashed tenant ID
    "nxa.tenant.tier": "enterprise|professional|starter",
    
    # Request context
    "nxa.request.id": "uuid",                    # Correlation ID
    "nxa.user.id": "hashed_user_id",             # Hashed for PII
    "nxa.user.role": "admin|viewer|operator",
    
    # Business context
    "nxa.asset.count": 15420,                    # For sync spans
    "nxa.sync.type": "full|incremental|delta|webhook",
    "nxa.sync.source": "lansweeper|intune|jamf|manual",
    
    # Error context (on error spans)
    "nxa.error.code": "ASSET_NOT_FOUND|SYNC_CONFLICT|LICENSE_EXCEEDED",
    "nxa.error.category": "validation|external|internal|transient",
    "nxa.error.retryable": "true|false",
}
```

### 7.4 Sampling Strategy

| Sampler | Rate | Criteria | Target |
|---------|------|----------|--------|
| **Parent-Based** | 100% | If parent sampled | Continue trace |
| **TraceID Ratio** | 10% | Default for all services | Cost control |
| **Rate-Limiting** | 1000/sec | Per service instance | Prevent overload |
| **Rule-Based** | 100% | Errors (status >= 500) | Debug failures |
| **Rule-Based** | 100% | Latency P99 > threshold | Debug slowness |
| **Rule-Based** | 100% | Tenant in `premium` tier | Premium SLA |
| **Rule-Based** | 100% | `x-debug-trace` header | Developer debugging |

```yaml
# otel-collector-config.yaml (sampling processor)
processors:
  tail_sampling:
    decision_wait: 30s
    num_traces: 50000
    expected_new_traces_per_sec: 1000
    policies:
      - name: "errors"
        type: "string_attribute"
        string_attribute:
          key: "error"
          values: ["true"]
      - name: "high_latency"
        type: "latency"
        latency:
          threshold_ms: 2000
      - name: "premium_tenant"
        type: "string_attribute"
        string_attribute:
          key: "nxa.tenant.tier"
          values: ["enterprise", "professional"]
      - name: "debug_header"
        type: "string_attribute"
        string_attribute:
          key: "http.request.header.x-debug-trace"
          values: ["true"]
      - name: "probabilistic"
        type: "probabilistic"
        probabilistic:
          sampling_percentage: 10
```

### 7.5 Trace Query Examples (Tempo/TraceQL)

```traceql
# Find slow asset sync traces for tenant
{span.service.name="nxa-sync-worker" && span.name="job.sync.full_asset_sync" && span.duration > 10m && nxa.tenant.id="tnt_abc123"}

# Find error traces with license exceeded
{span.status=error && nxa.error.code="LICENSE_EXCEEDED"}

# Find traces with DB query > 5s
{span.name=~"postgresql .*" && span.duration > 5s}

# Cross-service trace for asset detail view
{span.service.name="nxa-bff" && span.name="GET /api/v1/assets/:id" && span.trace_id="a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4"}
```

---

## 8. Log Aggregation (Loki)

### 8.1 Log Pipeline Architecture

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            LOG AGGREGATION PIPELINE                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  APPLICATION PODS                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                            │
│  │ nxa-api     │  │ nxa-worker  │  │ nxa-bff     │  ...                       │
│  │ (stdout)    │  │ (stdout)    │  │ (stdout)    │                            │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                            │
│         │                │                │                                    │
│         └────────────────┼────────────────┘                                    │
│                          ▼                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                    OPENTELEMETRY COLLECTOR (Filelog Receiver)           │   │
│  │  ┌─────────────────────────────────────────────────────────────────┐    │   │
│  │  │  INPUT: Container logs (/var/log/pods/*/*.log)                  │    │   │
│  │  │  PARSERS: JSON → structured, Regex → fallback                   │    │   │
│  │  │  PROCESSORS:                                                    │    │   │
│  │  │    • Add k8s metadata (pod, namespace, container, labels)       │    │   │
│  │  │    • Add resource attributes (service, version, tenant)         │    │   │
│  │  │    • Transform: redact PII (emails, IPs, tokens)                │    │   │
│  │  │    • Filter: drop health check logs (level=debug, path=/health) │    │   │
│  │  │    • Batch: 1000 logs or 10s                                    │    │   │
│  │  └─────────────────────────────────────────────────────────────────┘    │   │
│  └────────────────────────────────┬────────────────────────────────────────┘   │
│                                   │                                             │
│                                   ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                            LOKI (Distributed)                           │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │   │
│  │  │  Distributor │→ │   Ingester   │→ │   S3/MinIO   │  (TSDB blocks)   │   │
│  │  │  (HA, LB)    │  │  (HA, WAL)   │  │  (chunks)    │                  │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                  │   │
│  │        │                                    │                           │   │
│  │        ▼                                    ▼                           │   │
│  │  ┌──────────────┐                   ┌──────────────┐                  │   │
│  │  │   Querier    │◀──────────────────│  Compactor   │  (merge, TTL)    │   │
│  │  │  (parallel)  │                   │  (retention) │                  │   │
│  │  └──────────────┘                   └──────────────┘                  │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Log Labels (Low Cardinality Only)

| Label | Values | Purpose |
|-------|--------|---------|
| `namespace` | `nxa-prod`, `nxa-staging`, `nxa-dev`, `observability` | K8s namespace |
| `service` | `nxa-api`, `nxa-bff`, `nxa-worker`, `nxa-sync`, `traefik` | Service name |
| `pod` | Pod name (hashed) | Pod identification |
| `container` | `app`, `sidecar`, `init` | Container name |
| `level` | `debug`, `info`, `warn`, `error`, `critical` | Log level |
| `tenant_tier` | `enterprise`, `professional`, `starter` | Tenant tier (not ID) |

> **Warning:** Never label by `tenant_id`, `user_id`, `request_id`, `trace_id` — use structured metadata in log line instead.

### 8.3 LogQL Queries

```logql
# Error logs for API service in last hour
{namespace="nxa-prod", service="nxa-api", level=~"error|critical"} |= "tenant_id" | json | tenant_id="tnt_abc123"

# Sync job failures with context
{namespace="nxa-prod", service="nxa-sync-worker"} |= "sync_id" | json | level="error" | sync_type="full"

# Slow DB queries (>1s)
{namespace="nxa-prod", service="nxa-api"} |= "duration_ms" | json | duration_ms > 1000

# Auth failures by tenant tier
{namespace="nxa-prod", service="nxa-api"} |= "auth_failed" | json | tenant_tier="enterprise"

# Trace ID correlation
{namespace="nxa-prod"} |= "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4"
```

### 8.4 Log Retention & Cost Optimization

| Tier | Retention | Storage Class | Compression | Estimated Cost |
|------|-----------|---------------|-------------|----------------|
| **Hot** | 30 days | SSD (gp3) | ZSTD 3 | $0.10/GiB/mo |
| **Warm** | 90 days | S3 Standard-IA | ZSTD 3 | $0.0125/GiB/mo |
| **Cold** | 13 months | S3 Glacier IR | ZSTD 3 | $0.004/GiB/mo |

> **Tip:** Loki compactor merges chunks and applies retention. Configure `limits_config.retention_period: 395d` (13 months).

---

## 9. Real User Monitoring (RUM)

### 9.1 RUM Architecture

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              RUM DATA FLOW                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  BROWSER (Next.js App)                                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  @nexusasset/rum-agent (lightweight, ~2KB gzipped)                      │   │
│  │  • Auto-instruments: Navigation, Resource, Paint, Long Task, Errors    │   │
│  │  • Web Vitals: FCP, LCP, CLS, FID/INP, TTFB                            │   │
│  │  • Custom: Route changes, API calls, User interactions                 │   │
│  │  • Sampling: 10% (configurable per tenant tier)                        │   │
│  │  • Batch: 50 events or 5s → Beacon API → /rum/collect                  │   │
│  └────────────────────────────────┬────────────────────────────────────────┘   │
│                                   │                                             │
│                                   ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                    BFF RUM ENDPOINT (/rum/collect)                      │   │
│  │  • Validates payload, enriches with tenant/session context             │   │
│  │  • Forwards to OTLP/HTTP → Otel Collector → Tempo (as spans)           │   │
│  │  • Also writes to Loki (structured logs) for log-based analysis        │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 9.2 Core Web Vitals SLIs

| Metric | Good | Needs Improvement | Poor | SLO Target |
|--------|------|-------------------|------|------------|
| **FCP** (First Contentful Paint) | ≤ 1.8s | 1.8–3.0s | > 3.0s | 95% ≤ 1.8s (SLO-RB-008) |
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5–4.0s | > 4.0s | 95% ≤ 2.5s |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1–0.25 | > 0.25 | 95% ≤ 0.1 |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200–500ms | > 500ms | 95% ≤ 200ms |
| **TTFB** (Time to First Byte) | ≤ 800ms | 800–1800ms | > 1800ms | 95% ≤ 800ms |

### 9.3 RUM Dashboards

| Dashboard | Key Visualizations |
|-----------|-------------------|
| `rum-core-web-vitals` | FCP/LCP/CLS/INP distributions, geo heatmap, device breakdown |
| `rum-page-performance` | Route-level vitals, resource waterfall, long task analysis |
| `rum-api-performance` | BFF API latency from client perspective, error rates |
| `rum-user-journeys` | Funnel conversion, drop-off points, session replay links |
| `rum-tenant-comparison` | Per-tenant vitals, tier comparison, SLO compliance |

---

## 10. OpenTelemetry Collector Configuration

### 10.1 Collector Deployment Modes

| Mode | Replicas | Use Case | Resources |
|------|----------|----------|-----------|
| **DaemonSet** | 1 per node | Node-level log/metric collection, host metrics | 100m CPU, 128Mi Mem |
| **Deployment (Gateway)** | 3+ per AZ | Central trace/metric processing, sampling, export | 500m CPU, 512Mi Mem |
| **Sidecar** | Per pod | Legacy apps, custom instrumentation | 50m CPU, 64Mi Mem |

### 10.2 Core Configuration (Production)

```yaml
# otel-collector-gateway-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
  prometheus:
    config:
      scrape_configs:
        - job_name: 'kubernetes-pods'
          kubernetes_sd_configs:
            - role: pod
          relabel_configs:
            - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
              action: keep
              regex: true
  filelog:
    include: [/var/log/pods/*/*/*.log]
    start_at: beginning
    include_file_path: true
    include_file_name: false

processors:
  # Resource detection (k8s, cloud, host)
  resourcedetection:
    detectors: [k8snode, env, system]
    k8snode:
      auth_type: "serviceAccount"
    system:
      hostname_sources: ["os"]
  
  # Batch for throughput
  batch:
    timeout: 10s
    send_batch_size: 1024
    send_batch_max_size: 2048
  
  # Memory limiter (prevent OOM)
  memory_limiter:
    check_interval: 1s
    limit_mib: 1024
    spike_limit_mib: 256
  
  # Transform: normalize, redact, enrich
  transform:
    trace_statements:
      - context: span
        statements:
          - set(attributes["service.namespace"], "nexusasset")
          - set(attributes["deployment.environment"], getenv("ENVIRONMENT"))
    metric_statements:
      - context: datapoint
        statements:
          - set(description, "NexusAsset Cloud metric")
    log_statements:
      - context: log
        statements:
          - set(attributes["nxa.log.source"], "filelog")
  
  # Tail sampling (gateway only)
  tailsampling:
    decision_wait: 30s
    num_traces: 50000
    expected_new_traces_per_sec: 1000
    policies:
      - name: errors
        type: string_attribute
        string_attribute: {key: "error", values: ["true"]}
      - name: high_latency
        type: latency
        latency: {threshold_ms: 2000}
      - name: premium_tenant
        type: string_attribute
        string_attribute: {key: "nxa.tenant.tier", values: ["enterprise", "professional"]}
      - name: probabilistic
        type: probabilistic
        probabilistic: {sampling_percentage: 10}
  
  # Redact PII
  filter:
    logs:
      exclude:
        match_type: regexp
        match_expression: '(?i)(password|token|secret|api[_-]?key|authorization|credit[_-]?card|ssn)'

exporters:
  prometheusremotewrite:
    endpoint: "http://prometheus:9090/api/v1/write"
    external_labels:
      cluster: "${CLUSTER_NAME}"
      region: "${AWS_REGION}"
  tempo:
    endpoint: "tempo-distributor:4317"
    tls:
      insecure: true
  loki:
    endpoint: "http://loki-distributor:3100/loki/api/v1/push"
  logging:
    loglevel: debug
    sampling_initial: 5
    sampling_thereafter: 200

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [resourcedetection, batch, memory_limiter, tailsampling, transform, filter]
      exporters: [tempo, logging]
    metrics:
      receivers: [otlp, prometheus]
      processors: [resourcedetection, batch, memory_limiter, transform]
      exporters: [prometheusremotewrite, logging]
    logs:
      receivers: [otlp, filelog]
      processors: [resourcedetection, batch, memory_limiter, transform, filter]
      exporters: [loki, logging]
```

---

## 11. Cost Optimization

### 11.1 Observability Cost Breakdown (Monthly Estimate)

| Component | Data Volume | Monthly Cost | % of Total |
|-----------|-------------|--------------|------------|
| **Prometheus/Thanos** | 50M samples/min | $2,500 | 25% |
| **Tempo (Traces)** | 2M spans/min | $3,000 | 30% |
| **Loki (Logs)** | 100 GB/day | $2,000 | 20% |
| **Grafana Cloud/Enterprise** | — | $1,500 | 15% |
| **Data Transfer/Storage** | — | $1,000 | 10% |
| **Total** | — | **$10,000** | 100% |

### 11.2 Cost Control Strategies

| Strategy | Implementation | Savings |
|----------|----------------|---------|
| **Adaptive Sampling** | Tail sampling + probabilistic (10%) | 60% trace reduction |
| **Metric Cardinality Limits** | Drop labels > 100 values, relabel high-card | 40% metric reduction |
| **Log Level Control** | Default `info`, `debug` opt-in via configmap | 50% log reduction |
| **Retention Tiering** | Hot/Warm/Cold with S3 lifecycle | 70% storage savings |
| **Compression** | ZSTD for logs, Gorilla for metrics | 3-5x compression |
| **Tenant-Based Quotas** | Per-tenant ingestion limits | Prevents noisy neighbors |

---

## 12. Operational Procedures

### 12.1 On-Call Runbooks (Linked)

| Runbook | Trigger | Owner | Link |
|---------|---------|-------|------|
| **SLO Burn Rate** | `critical_api_availability_burn_rate_1h` | Platform | [runbooks.nexusasset.com/slo-api-availability](https://runbooks.nexusasset.com/slo-api-availability) |
| **Database Primary Down** | `critical_database_primary_down` | Platform | [runbooks.nexusasset.com/db-primary-down](https://runbooks.nexusasset.com/db-primary-down) |
| **Sync Job Failures** | `high_sync_job_failure_rate` | Backend | [runbooks.nexusasset.com/sync-failures](https://runbooks.nexusasset.com/sync-failures) |
| **K8s Control Plane** | `critical_kubernetes_control_plane_down` | Platform | [runbooks.nexusasset.com/k8s-control-plane](https://runbooks.nexusasset.com/k8s-control-plane) |
| **Trace Ingestion Lag** | `high_tempo_ingestion_lag` | Platform | [runbooks.nexusasset.com/tempo-lag](https://runbooks.nexusasset.com/tempo-lag) |
| **Log Ingestion Lag** | `high_loki_ingestion_lag` | Platform | [runbooks.nexusasset.com/loki-lag](https://runbooks.nexusasset.com/loki-lag) |

### 12.2 Incident Response Flow

```ascii
ALERT FIRES
    │
    ▼
┌─────────────────┐
│ PagerDuty pages │
│  on-call (P1)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ On-call joins   │
│ #incidents Slack│
│ + Zoom bridge   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Assess:         │
│ • SLO impact?   │
│ • Customer vis? │
│ • Blast radius? │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
   YES        NO
    │         │
    ▼         ▼
┌────────┐ ┌─────────────────┐
│ Declare│ │ Investigate via │
│ SEV-1  │ │ Grafana/Logs/   │
│ Page   │ │ Traces          │
│ Stake- │ │                 │
│ holders│ └────────┬────────┘
└────┬────┘          │
     │               ▼
     │        ┌─────────────────┐
     │        │ Root cause?     │
     │        └────────┬────────┘
     │             ┌───┴───┐
     │            YES     NO
     │             │       │
     ▼             ▼       ▼
┌────────┐   ┌──────────┐ ┌──────────────┐
│ Mitigate│   │ Workaround│ │ Escalate to  │
│ + Fix  │   │ + Monitor │ │ SME / Vendor │
└────────┘   └──────────┘ └──────────────┘
     │
     ▼
┌─────────────────┐
│ Verify SLO      │
│ recovery        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Resolve incident│
│ Postmortem if   │
│ SEV-1/2 or >30m │
└─────────────────┘
```

---

## 13. Compliance & Audit

### 13.1 Observability Data Retention for Compliance

| Regulation | Requirement | Implementation |
|------------|-------------|----------------|
| **SOC 2 Type II** | 12 months audit logs | Loki cold tier 13 months, immutable |
| **ISO 27001** | Access logging, monitoring | All admin actions traced, alerted |
| **GDPR** | Right to erasure | PII redaction in pipeline, tenant deletion workflow |
| **HIPAA** (future) | 6 years audit trail | Extended retention tier, encryption at rest |

### 13.2 Audit Log Requirements

| Event Category | Logged Fields | Retention |
|----------------|---------------|-----------|
| **Authentication** | user_id, tenant_id, method, ip, success, mfa | 13 months |
| **Authorization** | user_id, resource, action, decision, policy | 13 months |
| **Data Access** | user_id, tenant_id, table, query_hash, rows | 13 months |
| **Config Changes** | user_id, resource, before, after, git_sha | 13 months |
| **Infrastructure** | actor, action, target, result, timestamp | 13 months |

---

## 14. Future Enhancements (Roadmap)

| Quarter | Initiative | Description |
|---------|------------|-------------|
| **Q1 2027** | **Mimir Migration** | Replace Thanos/Prometheus with Grafana Mimir for unified metrics |
| **Q2 2027** | **eBPF Auto-Instrumentation** | Cilium/Hubble for kernel-level network/service observability |
| **Q2 2027** | **Continuous Profiling** | Pyroscope integration for CPU/memory/lock profiling |
| **Q3 2027** | **ML-Based Anomaly Detection** | Grafana ML / Cortex for seasonal anomaly detection |
| **Q3 2027** | **Session Replay** | Full DOM replay for frontend debugging (privacy-compliant) |
| **Q4 2027** | **Unified Query Language** | Single query across metrics/logs/traces (PromQL + LogQL + TraceQL) |

---

## 15. References

| Doc ID | Title | Link |
|--------|-------|------|
| DOC-05 | Application Architecture | [05-application-architecture.md](../00-overview/05-application-architecture.md) |
| DOC-06 | Infrastructure Architecture | [06-infrastructure-architecture.md](../00-overview/06-infrastructure-architecture.md) |
| DOC-07 | API Design | [07-api-design.md](../00-overview/07-api-design.md) |
| DOC-10 | Testing Strategy | [10-testing-strategy.md](../00-overview/10-testing-strategy.md) |
| DOC-11 | DevOps & CI/CD | [11-devops-cicd.md](../00-overview/11-devops-cicd.md) |
| ADR-019 | Observability Stack Selection | [ADR-019](../14-appendices/adr-index.md#adr-019) |
| ADR-020 | Sampling Strategy | [ADR-020](../14-appendices/adr-index.md#adr-020) |
| ADR-021 | SLO Framework | [ADR-021](../14-appendices/adr-index.md#adr-021) |
| ADR-022 | Log Redaction | [ADR-022](../14-appendices/adr-index.md#adr-022) |
| ADR-023 | RUM Implementation | [ADR-023](../14-appendices/adr-index.md#adr-023) |
| ADR-024 | Cost Optimization | [ADR-024](../14-appendices/adr-index.md#adr-024) |

---

> **Document Control:** This document is maintained by Platform Engineering. Changes require review by SRE Lead and Architecture Review Board. Next review: 2027-02-09.