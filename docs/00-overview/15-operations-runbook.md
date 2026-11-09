# DOC-15: Operations Runbook

---

| **Document ID** | DOC-15 |
|-----------------|--------|
| **Title** | Operations Runbook |
| **Version** | 1.0.0 |
| **Status** | ✓ Approved |
| **Owner** | Platform Engineering / SRE |
| **Last Updated** | 2026-11-09 |
| **Classification** | Internal — Confidential |
| **Related Docs** | [DOC-06 Infrastructure Architecture](../00-overview/06-infrastructure-architecture.md), [DOC-11 DevOps & CI/CD](../00-overview/11-devops-cicd.md), [DOC-12 Security Architecture](../00-overview/12-security-architecture.md), [DOC-13 Observability](../00-overview/13-observability.md), [DOC-14 Data Migration](../00-overview/14-data-migration-seed-strategy.md) |
| **Traceability** | REQ-NFR-OP-###, SLO/RB-###, ADR-031..036 |

---

## 1. Executive Summary

This **Operations Runbook** provides executable procedures for **Day 2 operations** of NexusAsset Cloud. It covers **incident response**, **on-call responsibilities**, **common operational tasks**, **disaster recovery**, **capacity management**, and **security operations**. All runbooks are **version-controlled**, **tested quarterly** via fire drills, and **integrated with alerting** for one-click access.

> **Note:** Runbooks are living documents. Update after every incident retrospective. Last fire drill: 2026-10-15. Next scheduled: 2027-01-15.

---

## 2. On-Call Organization

### 2.1 Rotation Schedule

| Role | Schedule | Coverage | Escalation | Tools |
|------|----------|----------|------------|-------|
| **Primary On-Call** | Weekly (Mon-Sun) | 24/7 | Secondary (15 min) | PagerDuty, Slack, Zoom |
| **Secondary On-Call** | Weekly (Mon-Sun) | 24/7 | Manager (30 min) | PagerDuty, Slack |
| **Platform Lead** | Business hours | Advisory | Director (1 hr) | Slack, Email |
| **Security On-Call** | Daily rotation | 24/7 (security alerts) | CISO (15 min) | PagerDuty, Slack |

### 2.2 On-Call Handoff Checklist

```markdown
# Weekly On-Call Handoff (Monday 09:00 UTC)

## Outgoing On-Call Completes:
- [ ] Update PagerDuty schedule with notes
- [ ] Post handoff summary in #oncall-handoff Slack
- [ ] Review open incidents from past week
- [ ] Document any pending actions / watch items
- [ ] Confirm runbook access for incoming

## Incoming On-Call Reviews:
- [ ] Read handoff summary
- [ ] Check PagerDuty for acknowledged but unresolved alerts
- [ ] Review Grafana "Active Alerts" dashboard
- [ ] Verify laptop/VPN access to production
- [ ] Confirm runbook URLs bookmarked
- [ ] Test Zoom bridge link
- [ ] Review upcoming deployments (ArgoCD)

## Handoff Template:
```
**Week of 2026-11-09**
**Outgoing:** @alice | **Incoming:** @bob

**Active Incidents:**
- INC-2026-045: Lansweeper sync lag (tenant tnt_abc123) - monitoring, 2h old
- INC-2026-046: False positive WAF alerts - tuning in progress

**Watch Items:**
- Tenant tnt_xyz789 approaching license limit (85%)
- Staging deployment v1.4.0 scheduled Tue 14:00 UTC
- Cert expiry: *.nexusasset.com in 14 days

**Runbook Updates:**
- Updated: runbooks/incident/db-primary-down (new failover steps)
- New: runbooks/incident/license-limit-exceeded
```
```

---

## 3. Incident Response Framework

### 3.1 Severity Classification

| Severity | Label | Definition | Response Target | Communication |
|----------|-------|------------|-----------------|---------------|
| **SEV-1** | Critical | **Customer-facing outage** (SLO breach, data loss, security breach) | **5 min** page, 15 min triage | StatusPage, Slack #incidents, Email stakeholders, Zoom bridge |
| **SEV-2** | Major | **Degraded functionality** (partial outage, performance < SLO, single tenant down) | **15 min** page, 30 min triage | Slack #incidents, StatusPage (optional), Zoom bridge |
| **SEV-3** | Minor | **Non-customer impacting** (internal tooling, non-prod, scheduled maintenance issue) | **1 hour** Slack, next business day | Slack #incidents |
| **SEV-4** | Low | **Observational** (capacity warning, cert expiry > 7d, log noise) | **24 hours** | Slack #alerts-info |

### 3.2 Incident Lifecycle

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        INCIDENT LIFECYCLE                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ALERT FIRES                                                                    │
│       │                                                                         │
│       ▼                                                                         │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │  TRIAGE     │───▶│  INVESTIGATE│───▶│  MITIGATE   │───▶│  RESOLVE    │      │
│  │  (0-15 min) │    │  (15-60 min)│    │  (ongoing)  │    │             │      │
│  └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘      │
│       │                   │                   │                   │              │
│       ▼                   ▼                   ▼                   ▼              │
│  • Acknowledge       • Identify          • Apply fix       • Verify fix       │
│  • Assess severity   • Root cause        • Workaround      • Close incident   │
│  • Declare SEV       • Impact assessment • Communicate     • Postmortem if    │
│  • Page stakeholders • Runbook exec      • Update status   │   SEV-1/2        │
│  • Create incident   • Engage SMEs       • Monitor         • Schedule retro   │
│  • Start Zoom bridge • Document          • Escalate        • Update runbooks  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 3.3 Incident Command Structure

| Role | Responsibilities | SEV-1 | SEV-2 | SEV-3 |
|------|------------------|-------|-------|-------|
| **Incident Commander (IC)** | Overall ownership, decisions, communication | Primary On-Call | Primary On-Call | Primary On-Call |
| **Technical Lead (TL)** | Technical investigation, fix implementation | Secondary On-Call | Secondary On-Call | Assigned Engineer |
| **Communications Lead (CL)** | Stakeholder updates, StatusPage, Slack | Platform Lead | Platform Lead | — |
| **Scribe** | Timeline documentation, action items | Rotating | Rotating | — |
| **SME** | Domain expertise (DB, K8s, Security, App) | As needed | As needed | As needed |

---

## 4. Core Runbooks (Linked from Alerts)

### 4.1 Runbook Index

| Runbook ID | Title | Trigger Alerts | Owner | Last Tested |
|------------|-------|----------------|-------|-------------|
| **RB-001** | [API Availability SLO Breach](../14-appendices/runbooks/rb-001-api-availability.md) | `critical_api_availability_burn_rate_1h` | Backend | 2026-10-15 |
| **RB-002** | [API Latency SLO Breach](../14-appendices/runbooks/rb-002-api-latency.md) | `critical_api_latency_p99_burn_rate_1h` | Backend | 2026-10-15 |
| **RB-003** | [Database Primary Down](../14-appendices/runbooks/rb-003-db-primary-down.md) | `critical_database_primary_down` | Platform | 2026-10-15 |
| **RB-004** | [Database Replication Lag](../14-appendices/runbooks/rb-004-db-replication-lag.md) | `high_database_replication_lag` | Platform | 2026-10-15 |
| **RB-005** | [Kubernetes Control Plane Down](../14-appendices/runbooks/rb-005-k8s-control-plane.md) | `critical_kubernetes_control_plane_down` | Platform | 2026-10-15 |
| **RB-006** | [Sync Job Failures](../14-appendices/runbooks/rb-006-sync-failures.md) | `high_sync_job_failure_rate` | Backend | 2026-10-15 |
| **RB-007** | [Sync Lag / Stalled](../14-appendices/runbooks/rb-007-sync-lag.md) | `high_sync_lag` | Backend | 2026-10-15 |
| **RB-008** | [Tempo Ingestion Lag](../14-appendices/runbooks/rb-008-tempo-lag.md) | `high_tempo_ingestion_lag` | Platform | 2026-10-15 |
| **RB-009** | [Loki Ingestion Lag](../14-appendices/runbooks/rb-009-loki-lag.md) | `high_loki_ingestion_lag` | Platform | 2026-10-15 |
| **RB-010** | [Certificate Expiry](../14-appendices/runbooks/rb-010-cert-expiry.md) | `warning_cert_expiry_30d` | Platform | 2026-10-15 |
| **RB-011** | [Disk Space Critical](../14-appendices/runbooks/rb-011-disk-space.md) | `critical_disk_space_90` | Platform | 2026-10-15 |
| **RB-012** | [License Limit Exceeded](../14-appendices/runbooks/rb-012-license-limit.md) | `high_tenant_license_usage_90` | Billing | 2026-10-15 |
| **RB-013** | [Security Incident - Auth Anomaly](../14-appendices/runbooks/rb-013-auth-anomaly.md) | `critical_auth_anomaly` | Security | 2026-10-15 |
| **RB-014** | [Security Incident - Data Exfiltration](../14-appendices/runbooks/rb-014-data-exfil.md) | `critical_data_exfiltration` | Security | 2026-10-15 |
| **RB-015** | [Ransomware / Encryption Event](../14-appendices/runbooks/rb-015-ransomware.md) | `critical_ransomware_detected` | Security | 2026-10-15 |

### 4.2 Runbook Template (Standard Format)

````markdown
# RB-XXX: Runbook Title

**Alert:** `alert_name`  
**Severity:** SEV-1/2/3  
**Owner:** @team  
**Last Updated:** YYYY-MM-DD  
**Last Tested:** YYYY-MM-DD (fire drill)

---

## 1. Quick Assessment (≤ 2 min)

| Check | Command / Dashboard | Expected | Action if Failed |
|-------|---------------------|----------|------------------|
| Service health | `https://grafana/d/svc-api-golden-signals` | All green | Proceed to investigation |
| Error budget | `https://grafana/d/slo-error-budget` | > 10% remaining | Declare SEV-1 |
| Recent deploy | `https://argocd/nexusasset` | No deploy in 30m | Check deploy correlation |

---

## 2. Investigation Steps

### Step 1: Check Dashboard
```bash
# Open dashboard
open "https://grafana/d/svc-api-golden-signals?var-region=all&var-tenant=all"

# Key panels to check:
# - Traffic (RPS) - sudden drop?
# - Errors (5xx rate) - spike?
# - Latency P99 - > 500ms?
# - Saturation - CPU/Mem > 85%?
```

### Step 2: Check Logs
```bash
# Recent errors (last 15 min)
kubectl logs -n nxa-prod -l app=nxa-api --since=15m | grep -i error

# Or via Loki
logcli query '{namespace="nxa-prod", service="nxa-api", level="error"} |= "trace_id"' --since=15m
```

### Step 3: Check Traces
```bash
# Find slow traces
tracecli query '{span.service.name="nxa-api" && span.duration > 5s}' --since=15m
```

---

## 3. Common Root Causes & Fixes

| Root Cause | Symptoms | Fix Command | Verification |
|------------|----------|-------------|--------------|
| **DB connection pool exhausted** | High latency, 503 errors, "pool timeout" logs | `kubectl scale deployment nxa-api --replicas=+2` | Latency P99 < 500ms |
| **Redis cache miss storm** | High DB load, increased latency | `kubectl exec redis-0 -- redis-cli FLUSHALL` (careful!) | Cache hit ratio > 95% |
| **Lansweeper API rate limit** | Sync job failures, 429 errors | Increase backoff, reduce concurrency | Sync success rate > 99% |
| **Bad deploy** | Errors correlate with deploy time | `argocd app rollback nexusasset-api <prev-version>` | Error rate returns to baseline |

---

## 4. Escalation Path

```
Primary On-Call (5 min)
    │
    ├── Cannot identify root cause in 15 min
    │       │
    │       ▼
    ├── Secondary On-Call (15 min)
    │       │
    │       ├── Requires DB admin / K8s expert
    │       │       │
    │       │       ▼
    │       │   Platform Lead (30 min)
    │       │       │
    │       │       ├── Customer impact confirmed
    │       │       │       │
    │       │       │       ▼
    │       │       │   Engineering Director (1 hr)
    │       │       │       │
    │       │       │       └── CTO / CEO (if prolonged)
    │       │       │
    │       │       └── Security incident → CISO (immediate)
```

---

## 5. Communication Templates

### Slack Incident Declaration
```
🚨 INCIDENT DECLARED: SEV-1
**Title:** API Availability SLO Breach
**Impact:** ~15% requests failing (5xx)
**Started:** 2026-11-09 14:32 UTC
**Commander:** @alice (Primary)
**Tech Lead:** @bob (Secondary)
**Bridge:** https://zoom.us/j/123456789
**StatusPage:** https://status.nexusasset.com/incidents/xyz
```

### StatusPage Update
```
**Investigating** - We are experiencing elevated API error rates affecting some customers. 
Our engineering team is investigating. We will provide an update within 15 minutes.
```

### Resolution Message
```
✅ INCIDENT RESOLVED: SEV-1
**Title:** API Availability SLO Breach
**Duration:** 47 minutes
**Root Cause:** Database connection pool exhaustion due to slow query
**Fix:** Scaled API replicas + query optimization deployed
**Postmortem:** Scheduled for 2026-11-11 10:00 UTC
```

---

## 6. Post-Incident Actions

- [ ] Close PagerDuty incident
- [ ] Update StatusPage to "Resolved"
- [ ] Schedule postmortem (SEV-1/2: within 48h; SEV-3: within 1 week)
- [ ] Create/runbook improvement tickets
- [ ] Update runbook with new findings
- [ ] Conduct fire drill if new scenario
````

---

## 5. Detailed Runbook: RB-003 Database Primary Down

````markdown
# RB-003: Database Primary Down

**Alert:** `critical_database_primary_down`  
**Severity:** SEV-1  
**Owner:** Platform Team  
**SLO Impact:** SLO-RB-010 (DB Write Availability 99.99%)  
**Runbook URL:** https://runbooks.nexusasset.com/rb-003-db-primary-down

---

## 1. Quick Assessment (≤ 2 min)

| Check | Command | Expected | Action |
|-------|---------|----------|--------|
| pg_up metric | `https://grafana/d/infra-postgresql-use` | `pg_up{role="primary"} == 1` | Proceed |
| Patroni status | `kubectl exec -n nxa-prod patroni-0 -- patronictl list` | Leader running | Proceed |
| Connection test | `pg_isready -h primary.nexusasset.internal -p 5432` | Accepting connections | Proceed |

---

## 2. Investigation

### Check Patroni Cluster State
```bash
# SSH to any Patroni pod
kubectl exec -n nxa-prod -it patroni-0 -- /bin/bash

# Check cluster state
patronictl list
# Expected output:
# + Cluster: nxa-db (1234567890) ---------+----+-----------+
# | Member | Host | Role | State | TL | Lag |
# +--------+------+------+-------+----+-----+
# | patroni-0 | 10.0.1.5 | Leader | running | 42 | 0 |
# | patroni-1 | 10.0.2.3 | Replica | running | 42 | 0 |
# | patroni-2 | 10.0.3.1 | Replica | running | 42 | 0 |
# +--------+------+------+-------+----+-----+

# If no Leader → failover needed
```

### Check PostgreSQL Logs
```bash
# Recent logs from primary
kubectl logs -n nxa-prod patroni-0 -c postgres --since=30m | tail -100

# Look for:
# - FATAL: could not connect to etcd
# - PANIC: could not write to WAL
# - ERROR: out of memory
# - CHECKPOINT: incomplete
```

---

## 3. Mitigation Procedures

### Scenario A: Patroni Leader Exists but Unreachable
```bash
# 1. Check network connectivity
kubectl exec -n nxa-prod patroni-0 -- ping primary.nexusasset.internal

# 2. Check PgBouncer
kubectl exec -n nxa-prod pgbouncer-0 -- pg_isready -p 6432

# 3. Restart PgBouncer (often fixes connection issues)
kubectl rollout restart deployment/pgbouncer -n nxa-prod

# 4. Verify
pg_isready -h primary.nexusasset.internal -p 6432
```

### Scenario B: No Patroni Leader (Failover Required)
```bash
# 1. Check etcd health (Patroni dependency)
kubectl exec -n nxa-prod etcd-0 -- etcdctl endpoint health

# 2. If etcd healthy, trigger manual failover
patronictl failover --cluster=nxa-db --candidate=patroni-1

# 3. Monitor failover
watch -n 5 'patronictl list'

# 4. Verify new primary
pg_isready -h primary.nexusasset.internal -p 5432
```

### Scenario C: Primary Pod Crashed (OOM / Disk Full)
```bash
# 1. Check pod status
kubectl get pods -n nxa-prod -l app=patroni

# 2. If OOMKilled, check memory limits
kubectl describe pod patroni-0 -n nxa-prod | grep -A 5 Limits

# 3. Increase memory limit if needed
kubectl patch statefulset patroni -n nxa-prod -p \
  '{"spec":{"template":{"spec":{"containers":[{"name":"postgres","resources":{"limits":{"memory":"8Gi"}}}]}}}}'

# 4. Delete pod to trigger restart
kubectl delete pod patroni-0 -n nxa-prod

# 5. Patroni will re-elect leader automatically
```

### Scenario D: Disk Full on Primary
```bash
# 1. Check disk usage
kubectl exec -n nxa-prod patroni-0 -- df -h /var/lib/postgresql/data

# 2. If > 90%, clean up:
#    - Remove old WAL files (Patroni manages retention)
#    - Check for runaway tables: SELECT pg_size_pretty(pg_total_relation_size('large_table'));
#    - Vacuum bloated tables: VACUUM FULL (requires lock)

# 3. Emergency: Add disk space (EBS volume expansion)
aws ec2 modify-volume --volume-id vol-xxx --size 1000

# 4. Restart PostgreSQL
kubectl exec -n nxa-prod patroni-0 -- patronictl restart patroni-0
```

---

## 4. Verification

```bash
# 1. Verify primary accepting writes
psql "postgresql://app:pass@primary.nexusasset.internal:5432/nexusasset" -c "INSERT INTO test_write (data) VALUES ('failover_test');"

# 2. Verify replication
psql "postgresql://app:pass@primary.nexusasset.internal:5432/nexusasset" -c "SELECT * FROM pg_stat_replication;"

# 3. Check application health
curl -f https://api.nexusasset.com/health/db

# 4. Monitor error budget recovery
# Grafana: slo-error-budget dashboard
```

---

## 5. Post-Incident

- [ ] Document timeline in incident tracker
- [ ] Root cause analysis (5 Whys)
- [ ] Update capacity planning if resource exhaustion
- [ ] Review Patroni/etcd configuration
- [ ] Schedule fire drill for failover scenario
````

---

## 6. Operational Procedures

### 6.1 Deployment Operations

#### Blue/Green Deploy (Standard)
```bash
# 1. Deploy to inactive environment (Blue)
kubectl set image deployment/nxa-api-blue nxa-api=nexusasset/api:v1.4.0 -n nxa-prod-blue

# 2. Run migrations on Blue
kubectl exec -n nxa-prod-blue deploy/nxa-api-blue -- ./manage.py migrate --noinput

# 3. Health check Blue
./scripts/health_check.sh blue

# 4. Canary: 5% traffic to Blue
./scripts/update_traffic_split.sh blue 5 green 95

# 5. Monitor 10 min
./scripts/monitor_canary.sh --duration=600

# 6. Progressive rollout: 25% → 50% → 75% → 100%
for split in 25 50 75 100; do
  ./scripts/update_traffic_split.sh blue $split green $((100-split))
  ./scripts/monitor_canary.sh --duration=300
done

# 7. Decommission Green
kubectl scale deployment/nxa-api-green --replicas=0 -n nxa-prod-green
```

#### Rollback Procedure
```bash
# Emergency rollback (< 2 min)
./scripts/emergency_rollback.sh --target=green --reason="SEV-1: API 5xx spike"

# Manual rollback steps:
# 1. Route 100% to previous environment
./scripts/update_traffic_split.sh green 100 blue 0

# 2. Verify health
./scripts/health_check.sh green

# 3. Scale down failed environment
kubectl scale deployment/nxa-api-blue --replicas=0 -n nxa-prod-blue
```

### 6.2 Certificate Management

```bash
# Check certificate expiry
kubectl get certificates -A -o custom-columns="NAME:.metadata.name,NAMESPACE:.metadata.name,EXPIRY:.status.notAfter"

# Renew Let's Encrypt cert (cert-manager)
kubectl delete secret tls-nexusasset-prod -n nxa-prod  # Forces renewal

# Manual cert replacement
kubectl create secret tls tls-nexusasset-prod \
  --cert=fullchain.pem --key=privkey.pem \
  -n nxa-prod --dry-run=client -o yaml | kubectl apply -f -

# Restart Traefik to pick up new cert
kubectl rollout restart deployment/traefik -n traefik
```

### 6.3 Capacity Management

```bash
# Check cluster capacity
kubectl top nodes --sort-by=memory
kubectl top pods -A --sort-by=memory | head -30

# Check PVC usage
kubectl get pvc -A -o custom-columns="NAME:.metadata.name,NAMESPACE:.metadata.name,CAPACITY:.spec.resources.requests.storage,USED:.status.capacity.storage"

# Scale node pool (AWS EKS)
aws eks update-nodegroup-config --cluster-name nxa-prod --nodegroup-name general --scaling-config minSize=10,maxSize=50,desiredSize=20

# Scale specific deployment
kubectl scale deployment nxa-api --replicas=20 -n nxa-prod
```

### 6.4 Database Operations

```bash
# Manual backup
kubectl exec -n nxa-prod patroni-0 -c postgres -- pg_dump -Fc nexusasset > backup_$(date +%Y%m%d).dump

# Restore from backup
kubectl exec -n nxa-prod patroni-0 -c postgres -- pg_restore -d nexusasset backup_20261109.dump

# Vacuum analyze (maintenance window)
kubectl exec -n nxa-prod patroni-0 -c postgres -- psql -d nexusasset -c "VACUUM ANALYZE;"

# Reindex bloated indexes
kubectl exec -n nxa-prod patroni-0 -c postgres -- psql -d nexusasset -c "REINDEX TABLE CONCURRENTLY nxa_assets_asset;"

# Check for long-running queries
kubectl exec -n nxa-prod patroni-0 -c postgres -- psql -d nexusasset -c "
  SELECT pid, now() - pg_stat_activity.query_start AS duration, query
  FROM pg_stat_activity
  WHERE state = 'active' AND now() - pg_stat_activity.query_start > interval '5 minutes';
"
```

### 6.5 Redis Operations

```bash
# Check memory usage
kubectl exec -n nxa-prod redis-0 -- redis-cli INFO memory | grep used_memory_human

# Flush cache (careful!)
kubectl exec -n nxa-prod redis-0 -- redis-cli FLUSHDB

# Check connected clients
kubectl exec -n nxa-prod redis-0 -- redis-cli CLIENT LIST

# Slow log
kubectl exec -n nxa-prod redis-0 -- redis-cli SLOWLOG GET 10

# Monitor real-time
kubectl exec -n nxa-prod redis-0 -- redis-cli MONITOR
```

### 6.6 Kafka Operations

```bash
# Check consumer lag
kubectl exec -n nxa-prod kafka-0 -- kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 --group nxa-sync-group --describe

# Reset consumer offset (emergency)
kubectl exec -n nxa-prod kafka-0 -- kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 --group nxa-sync-group \
  --reset-offsets --to-earliest --execute --topic nxa.asset.events

# Check disk usage
kubectl exec -n nxa-prod kafka-0 -- df -h /var/lib/kafka/data

# List topics
kubectl exec -n nxa-prod kafka-0 -- kafka-topics.sh --bootstrap-server localhost:9092 --list
```

---

## 7. Disaster Recovery

### 7.1 DR Scenarios & RTO/RPO

| Scenario | RTO | RPO | Recovery Procedure |
|----------|-----|-----|-------------------|
| **Single AZ Failure** | < 5 min | < 1 min | Auto-failover (multi-AZ), DNS update |
| **Region Failure** | < 30 min | < 5 min | Cross-region failover (manual trigger) |
| **Database Corruption** | < 1 hour | < 1 min | PITR from WAL-G + schema restore |
| **Kubernetes Cluster Loss** | < 2 hours | < 5 min | Recreate cluster from GitOps (ArgoCD) |
| **Ransomware / Data Destruction** | < 4 hours | < 24 hours | Isolate, restore from immutable backups, forensic |
| **Complete Account Compromise** | < 8 hours | N/A | Rotate all credentials, rebuild from clean state |

### 7.2 Region Failover Procedure

```bash
# 1. Declare DR event (SEV-1)
# IC creates incident, pages Platform Lead + Director

# 2. Verify primary region status
aws route53 get-health-check-status --health-check-id $PRIMARY_HC_ID

# 3. Update DNS to secondary region (manual approval required)
aws route53 change-resource-record-sets --hosted-zone-id $ZONE_ID --change-batch file://dr-failover.json

# dr-failover.json:
{
  "Changes": [{
    "Action": "UPSERT",
    "ResourceRecordSet": {
      "Name": "api.nexusasset.com",
      "Type": "CNAME",
      "TTL": 60,
      "SetIdentifier": "primary",
      "Failover": "PRIMARY",
      "ResourceRecords": [{"Value": "api-dr.nexusasset.com"}],
      "HealthCheckId": "$DR_HC_ID"
    }
  }]
}

# 4. Verify secondary region capacity
kubectl --context=dr-cluster top nodes
# Scale if needed

# 5. Verify database replication caught up
# Check replication lag < 1 min

# 6. Update StatusPage
# Communicate to customers

# 7. Document timeline
```

### 7.3 Point-in-Time Recovery (PITR)

```bash
# Restore database to specific timestamp
# Prerequisites: WAL-G configured, base backup exists

TARGET_TIME="2026-11-09T10:30:00Z"  # Before corruption
TARGET_DB="nexusasset"

# 1. Create recovery namespace
kubectl create namespace nxa-recovery

# 2. Restore base backup + WAL to target time
wal-g backup-fetch \
  --target-time="$TARGET_TIME" \
  --target-db="$TARGET_DB" \
  --namespace=nxa-recovery

# 3. Validate restored data
# Run integrity checks, row counts, checksums

# 4. Swap (requires downtime or blue/green)
# Update connection strings, restart apps

# 5. Verify application health
```

---

## 8. Security Operations

### 8.1 Security Incident Runbooks

| Runbook | Trigger | Key Actions |
|---------|---------|-------------|
| **RB-013** Auth Anomaly | Impossible travel, brute force, credential stuffing | Block IP, force password reset, revoke sessions, notify user |
| **RB-014** Data Exfiltration | Unusual data egress, DLP alerts | Isolate affected resources, preserve logs, legal notification |
| **RB-015** Ransomware | File encryption detected, ransom note | Isolate network, snapshot forensics, restore from immutable backup |

### 8.2 Credential Rotation

```bash
# Rotate database passwords (quarterly)
./scripts/rotate_db_password.sh --env=prod

# Rotate API keys (monthly)
./scripts/rotate_api_keys.sh --app=nxa-api --env=prod

# Rotate TLS certificates (90 days via cert-manager)
# Automatic, but verify:
kubectl get certificates -A -o wide | grep -v "True"

# Rotate service account tokens (K8s)
kubectl create token nxa-api-sa -n nxa-prod --duration=24h
```

### 8.3 Vulnerability Response

```bash
# CVE detected in base image
# 1. Check affected images
trivy image nexusasset/api:v1.4.0

# 2. Rebuild with patched base
# Update Dockerfile FROM line, rebuild, scan

# 3. Deploy to staging
argocd app set nexusasset-api --parameter=image.tag=v1.4.1

# 4. Validate, then promote to prod
```

---

## 9. Maintenance Windows

### 9.1 Scheduled Maintenance

| Window | Frequency | Duration | Activities | Approval |
|--------|-----------|----------|------------|----------|
| **Weekly** | Sunday 02:00-04:00 UTC | 2 hours | OS patches, minor DB vacuum, cert renewal | Auto (pre-approved) |
| **Monthly** | First Saturday 00:00-06:00 UTC | 6 hours | K8s version upgrade, major PG version, node rotation | Manual (Change Advisory Board) |
| **Quarterly** | Scheduled | 12 hours | DR test, failover drill, capacity review | Executive |

### 9.2 Maintenance Checklist

```markdown
# Pre-Maintenance (T-24h)
- [ ] Create maintenance window in StatusPage
- [ ] Notify customers (email + in-app banner)
- [ ] Confirm on-call coverage
- [ ] Verify backup completion
- [ ] Stage changes in GitOps (ArgoCD)
- [ ] Run pre-checks: `./scripts/pre_maintenance_checks.sh`

# During Maintenance
- [ ] Enable maintenance mode (Traefik 503 page)
- [ ] Execute changes per runbook
- [ ] Run validation tests
- [ ] Monitor error budget

# Post-Maintenance
- [ ] Disable maintenance mode
- [ ] Verify full health
- [ ] Update StatusPage
- [ ] Document any issues
- [ ] Close maintenance window
```

---

## 10. Communication Channels

| Channel | Purpose | Audience | Access |
|---------|---------|----------|--------|
| **#incidents** | Active incident coordination | On-call, SMEs, Leads | All engineering |
| **#oncall-handoff** | Weekly handoff notes | On-call rotation | On-call only |
| **#alerts-warning** | P3/P4 alerts | On-call, Team | All engineering |
| **#alerts-critical** | P1/P2 alerts (PagerDuty mirror) | On-call, Leads | On-call + Leads |
| **#platform-ops** | General ops discussion | Platform team | Platform team |
| **Zoom Bridge** | Incident war room | IC, TL, SMEs, CL | Link in incident |
| **StatusPage** | Customer communication | Customers, Public | Public |
| **PagerDuty** | Alert routing, on-call mgmt | On-call | On-call + Leads |

---

## 11. Runbook Testing & Improvement

### 11.1 Fire Drill Schedule

| Quarter | Scenario | Participants | Success Criteria |
|---------|----------|--------------|------------------|
| Q1 2027 | DB Primary Failover | Primary, Secondary, Platform Lead | Failover < 2 min, zero data loss |
| Q2 2027 | Region Failover (DR) | All on-call, Director | DNS switch < 15 min, app healthy |
| Q3 2027 | Ransomware Simulation | Security, Platform, Legal | Isolation < 5 min, restore < 4 hr |
| Q4 2027 | Complete Cluster Rebuild | Platform, SRE | GitOps restore < 2 hr |

### 11.2 Runbook Quality Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Runbook Coverage** | 100% of P1/P2 alerts | Alert-to-runbook mapping audit |
| **Runbook Accuracy** | 0 critical errors/fire drill | Fire drill retrospective |
| **Runbook Freshness** | < 90 days since update | Quarterly review |
| **Time to Mitigate (TTM)** | < 15 min (P1), < 60 min (P2) | Incident analytics |

---

## 12. References

| Doc ID | Title | Link |
|--------|-------|------|
| DOC-06 | Infrastructure Architecture | [06-infrastructure-architecture.md](../00-overview/06-infrastructure-architecture.md) |
| DOC-11 | DevOps & CI/CD | [11-devops-cicd.md](../00-overview/11-devops-cicd.md) |
| DOC-12 | Security Architecture | [12-security-architecture.md](../00-overview/12-security-architecture.md) |
| DOC-13 | Observability | [13-observability.md](../00-overview/13-observability.md) |
| DOC-14 | Data Migration | [14-data-migration-seed-strategy.md](../00-overview/14-data-migration-seed-strategy.md) |
| ADR-031 | Incident Response Process | [ADR-031](../14-appendices/adr-index.md#adr-031) |
| ADR-032 | On-Call Organization | [ADR-032](../14-appendices/adr-index.md#adr-032) |
| ADR-033 | DR Strategy | [ADR-033](../14-appendices/adr-index.md#adr-033) |
| ADR-034 | Maintenance Windows | [ADR-034](../14-appendices/adr-index.md#adr-034) |
| ADR-035 | Runbook Standards | [ADR-035](../14-appendices/adr-index.md#adr-035) |
| ADR-036 | Security Operations | [ADR-036](../14-appendices/adr-index.md#adr-036) |

---

> **Document Control:** This document is maintained by Platform Engineering / SRE. Changes require review by Platform Lead. Fire drills conducted quarterly. Next review: 2027-02-09.