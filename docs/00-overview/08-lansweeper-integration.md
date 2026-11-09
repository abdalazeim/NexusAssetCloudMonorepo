╔═════════════════════════════════════════════════════════════════════════════════════════════════╗
║ Document ID: DOC-08                                                                            ║
║ Title: Lansweeper Integration — GraphQL, Webhooks, Delta Sync, Field Ownership                 ║
║ Version: 1.0.0                                                                                 ║
║ Status: ✓ Approved                                                                             ║
║ Classification: Internal — Confidential                                                        ║
║ Authors: NexusAsset Cloud Integration Team                                                     ║
║ Reviewers: Backend, Security, Data Engineering, Platform                                       ║
║ Approved By: CTO / VP Engineering / Integration Architect                                      ║
║ Date: 2026-11-09                                                                               ║
║ Supersedes: —                                                                                  ║
║ Related: DOC-00, DOC-01, DOC-02, DOC-03, DOC-04, DOC-05, DOC-06, DOC-07, DOC-10, DOC-13       ║
║ Traceability: REQ-FN-4## ← CAP-011/CAP-016/CAP-061, UC-LS-###, DSV-006, EVT-LS-###, IEV-006..009 ║
╚════════════════════════════════════════════════════════════════════════════════════════════════╝

# ═══════════════════════════════════════════════════════════════
# NexusAsset Cloud — Lansweeper Integration (DOC-08)
# ═══════════════════════════════════════════════════════════════

> **Executive Summary** — This document specifies the deep integration between NexusAsset Cloud and Lansweeper, the primary on-premises IT asset discovery source. The integration uses **GraphQL API** for bulk/historical sync, **webhooks** for real-time delta updates, **idempotent reconciliation** with a **Field Ownership Matrix** for conflict resolution, and **software license/vulnerability correlation**. The design ensures **exactly-once processing**, **auditability**, and **tenant isolation**. All sync operations are observable via metrics, logs, and traces.

---

## Table of Contents

1. [Integration Overview](#1-integration-overview)
2. [Lansweeper GraphQL API](#2-lansweeper-graphql-api)
3. [Sync Architecture](#3-sync-architecture)
4. [Delta Sync & Webhooks](#4-delta-sync--webhooks)
5. [Reconciliation Engine](#5-reconciliation-engine)
6. [Field Ownership Matrix](#6-field-ownership-matrix)
7. [Conflict Resolution](#7-conflict-resolution)
8. [Software License Reconciliation](#8-software-license-reconciliation)
9. [Vulnerability Correlation](#9-vulnerability-correlation)
10. [Credential Management](#10-credential-management)
11. [Monitoring & Observability](#11-monitoring--observability)
12. [Error Handling & Retry](#12-error-handling--retry)
13. [Security Considerations](#13-security-considerations)
14. [Diagrams](#14-diagrams)
15. [Tables](#15-tables)

---

## 1. Integration Overview

### 1.1 Integration Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              LANSWEEPER INTEGRATION ARCHITECTURE                                    │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  LANSWEEPER ON-PREM                                                                                │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  Lansweeper Installation                                                                    │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                        │   │
│  │  │ Scanning    │  │ SQL Database│  │ GraphQL API │  │ Webhook     │                        │   │
│  │  │ Service     │──▶│ (Asset Data)│──▶│ (Query/Mut) │──▶│ Dispatcher  │                        │   │
│  │  └─────────────┘  └─────────────┘  └──────┬──────┘  └──────┬──────┘                        │   │
│  │                                            │               │                                │   │
│  │                                            │ HTTPS         │ HTTPS                          │   │
│  │                                            ▼               ▼                                │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
│       ┌───────────────────────────────────────────────────────────────────────────────────────┐    │
│       │                            NEXUSASSET CLOUD                                           │    │
│       │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐   │    │
│       │  │  GraphQL Client │  │  Webhook        │  │  Sync           │  │  Reconciliation │   │    │
│       │  │  (Polling)      │──▶│  Receiver       │──▶│  Orchestrator   │──▶│  Engine         │   │    │
│       │  │                 │  │  (Traefik)      │  │  (Celery)       │  │  (Field Matrix) │   │    │
│       │  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘   │    │
│       │           │                    │                    │                    │             │    │
│       │           ▼                    ▼                    ▼                    ▼             │    │
│       │  ┌─────────────────────────────────────────────────────────────────────────────────┐  │    │
│       │  │                        OUTBOX PATTERN (Kafka)                                   │  │    │
│       │  │  AssetDiscovered → AssetReconciled → AssetAuthorized → Integration Events       │  │    │
│       │  └─────────────────────────────────────────────────────────────────────────────────┘  │    │
│       │           │                    │                    │                    │             │    │
│       │           ▼                    ▼                    ▼                    ▼             │    │
│       │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐   │    │
│       │  │ nxa_assets      │  │ nxa_audit       │  │ nxa_billing     │  │ nxa_notifications│  │    │
│       │  │ (CMDB)          │  │ (Compliance)    │  │ (License/True-up)│  │ (Alerts)        │   │    │
│       │  └─────────────────┘  └─────────────────┘  └─────────────────┘  └─────────────────┘   │    │
│       └───────────────────────────────────────────────────────────────────────────────────────┘    │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Supported Lansweeper Versions

| Lansweeper Version | GraphQL API | Webhooks | Status |
|--------------------|-------------|----------|--------|
| 10.x (Current) | ✅ Full | ✅ Full | Supported |
| 9.x | ✅ Partial | ❌ | Legacy support |
| 8.x | ❌ | ❌ | Unsupported |

### 1.3 Asset Types Synchronized

| Lansweeper Type | NexusAsset Type | Sync Direction |
|-----------------|-----------------|----------------|
| Windows Computer | SERVER / LAPTOP | Lansweeper → NexusAsset |
| Linux Computer | SERVER | Lansweeper → NexusAsset |
| Mac Computer | LAPTOP | Lansweeper → NexusAsset |
| Network Device | NETWORK_DEVICE | Lansweeper → NexusAsset |
| VMware VM | VM | Lansweeper → NexusAsset |
| Hyper-V VM | VM | Lansweeper → NexusAsset |
| Monitor | PERIPHERAL | Lansweeper → NexusAsset |
| Printer | PERIPHERAL | Lansweeper → NexusAsset |
| Software | SOFTWARE (nested) | Lansweeper → NexusAsset |

---

## 2. Lansweeper GraphQL API

### 2.1 GraphQL Schema (Key Queries)

```graphql
# Asset query with pagination
query GetAssets(
  $first: Int!
  $after: String
  $filter: AssetFilter
  $orderBy: AssetOrderBy
) {
  assets(first: $first, after: $after, filter: $filter, orderBy: $orderBy) {
    pageInfo {
      hasNextPage
      endCursor
    }
    edges {
      cursor
      node {
        assetId
        assetName
        assetType
        serialNumber
        macAddress
        ipAddress
        manufacturer
        model
        cpu
        memory
        disk
        operatingSystem
        lastSeen
        lastChanged
        customFields {
          key
          value
        }
        software {
          name
          version
          publisher
          installDate
        }
      }
    }
  }
}

# Single asset by ID
query GetAsset($assetId: ID!) {
  asset(assetId: $assetId) {
    assetId
    assetName
    # ... all fields
  }
}

# Asset count for progress tracking
query GetAssetCount($filter: AssetFilter) {
  assetsAggregate(filter: $filter) {
    count
  }
}
```

### 2.2 GraphQL Client Configuration

```python
# backend/nexusasset/lansweeper/infrastructure/http/graphql_client.py
class LansweeperGraphQLClient:
    DEFAULT_PAGE_SIZE = 1000
    MAX_RETRIES = 3
    TIMEOUT = 30  # seconds
    
    def __init__(self, site: LansweeperSite, vault: CredentialVault):
        self.site = site
        self.vault = vault
        self.http_client = ResilientHttpClient(
            base_url=site.graphql_endpoint,
            timeout=TIMEOUT,
            retries=MAX_RETRIES,
            circuit_breaker=True,
        )
    
    async def get_credentials(self) -> LansweeperCredentials:
        """Retrieve and decrypt credentials from Vault"""
        secret_ref = self.site.auth_config.credential_ref
        return await self.vault.get_secret(secret_ref)
    
    async def query_assets_paginated(
        self, 
        cursor: Optional[str], 
        filter: AssetFilter,
        page_size: int = DEFAULT_PAGE_SIZE
    ) -> GraphQLAssetPage:
        """Execute paginated asset query with cursor"""
        credentials = await self.get_credentials()
        headers = {"Authorization": f"Bearer {credentials.access_token}"}
        
        variables = {
            "first": page_size,
            "after": cursor,
            "filter": filter.to_graphql(),
            "orderBy": {"field": "LAST_CHANGED", "direction": "ASC"}
        }
        
        response = await self.http_client.post(
            "/graphql",
            json={"query": ASSETS_QUERY, "variables": variables},
            headers=headers
        )
        return GraphQLAssetPage.from_response(response.json())
```

### 2.3 Rate Limiting & Throttling

| Limit | Value | Enforcement |
|-------|-------|-------------|
| **Concurrent Requests** | 5 per site | Client-side semaphore |
| **Requests/Second** | 10 | Token bucket (Redis) |
| **Daily Quota** | 100,000 assets | Tracked in sync job |
| **Backoff** | Exponential (1s, 2s, 4s, 8s, 16s) | On 429/5xx |

---

## 3. Sync Architecture

### 3.1 Sync Job Types

| Sync Type | Trigger | Scope | Frequency |
|-----------|---------|-------|-----------|
| **Full Sync** | Manual / Weekly scheduled | All assets in site | Weekly (Sunday 02:00 UTC) |
| **Incremental Sync** | Scheduled (cron) | Assets changed since last sync | Every 15 minutes |
| **Delta Sync** | Webhook | Single asset changed | Real-time |
| **Retry Sync** | Failed job retry | Failed assets only | Exponential backoff |

### 3.2 Sync Orchestrator (Celery Task)

```python
# backend/nexusasset/lansweeper/infrastructure/celery/sync_tasks.py
@celery_app.task(
    bind=True,
    max_retries=3,
    default_retry_delay=60,
    autoretry_for=(SyncRetryableError,),
    retry_backoff=True,
    retry_backoff_max=3600,
)
def execute_sync_job(self, sync_job_id: str) -> SyncJobResult:
    """Main sync execution entry point"""
    sync_job = SyncJobRepository.get(sync_job_id)
    site = LansweeperSiteRepository.get(sync_job.site_id)
    
    # Initialize sync context
    context = SyncContext(
        job_id=sync_job_id,
        site=site,
        sync_type=sync_job.sync_type,
        cursor=sync_job.cursor,
        idempotency_key=sync_job.idempotency_key,
    )
    
    try:
        if sync_job.sync_type == SyncType.FULL:
            result = FullSyncStrategy().execute(context)
        elif sync_job.sync_type == SyncType.INCREMENTAL:
            result = IncrementalSyncStrategy().execute(context)
        elif sync_job.sync_type == SyncType.DELTA:
            result = DeltaSyncStrategy().execute(context)
        
        # Update job status
        sync_job.complete(result)
        SyncJobRepository.update(sync_job)
        
        # Emit completion event
        EventPublisher.publish(SyncJobCompletedEvent(
            job_id=sync_job_id,
            site_id=site.id,
            tenant_id=site.tenant_id,
            stats=result.stats,
            status=SyncStatus.COMPLETED
        ))
        
        return result
    
    except Exception as e:
        sync_job.fail(str(e))
        SyncJobRepository.update(sync_job)
        
        EventPublisher.publish(SyncJobFailedEvent(
            job_id=sync_job_id,
            site_id=site.id,
            tenant_id=site.tenant_id,
            error=str(e),
            retryable=is_retryable(e)
        ))
        raise
```

### 3.3 Sync Strategies

#### Full Sync Strategy
```python
class FullSyncStrategy:
    def execute(self, context: SyncContext) -> SyncResult:
        stats = SyncStatistics()
        cursor = None
        
        while True:
            page = graphql_client.query_assets_paginated(
                cursor=cursor,
                filter=context.site.asset_type_filters,
                page_size=1000
            )
            
            for asset_edge in page.edges:
                try:
                    discovered_asset = self._normalize_asset(asset_edge.node)
                    reconciliation_result = ReconciliationEngine.reconcile(
                        discovered_asset, 
                        context.site.tenant_id
                    )
                    stats.increment(reconciliation_result)
                except Exception as e:
                    stats.record_error(asset_edge.node.assetId, str(e))
            
            stats.processed += len(page.edges)
            cursor = page.page_info.end_cursor
            
            if not page.page_info.has_next_page:
                break
            
            # Update job cursor for resumability
            SyncJobRepository.update_cursor(context.job_id, cursor)
        
        return SyncResult(stats=stats, final_cursor=cursor)
```

#### Incremental Sync Strategy
```python
class IncrementalSyncStrategy:
    def execute(self, context: SyncContext) -> SyncResult:
        # Use last successful sync timestamp
        since = context.site.last_sync_at or (utc_now() - timedelta(days=1))
        
        filter = AssetFilter(
            last_changed_gte=since,
            asset_types=context.site.asset_type_filters
        )
        
        return FullSyncStrategy().execute(
            SyncContext(..., filter=filter, cursor=context.cursor)
        )
```

---

## 4. Delta Sync & Webhooks

### 4.1 Webhook Registration

```http
POST /api/v1/lansweeper/sites/{site_id}/webhooks
{
  "events": [
    "asset.created",
    "asset.updated", 
    "asset.deleted",
    "asset.software.changed"
  ],
  "target_url": "https://api.nexusasset.cloud/api/v1/webhooks/lansweeper/{site_id}",
  "secret": "whsec_lansweeper_abc123...",
  "retry_policy": {
    "max_attempts": 5,
    "backoff": "exponential"
  }
}
```

### 4.2 Webhook Receiver (Traefik → nxa-backend)

```python
# backend/nexusasset/lansweeper/interfaces/webhooks/lansweeper_webhook.py
@router.post("/webhooks/lansweeper/{site_id}")
async def receive_lansweeper_webhook(
    site_id: str,
    request: Request,
    webhook_service: WebhookService = Depends()
):
    # Verify signature
    signature = request.headers.get("X-Lansweeper-Signature")
    payload = await request.body()
    
    if not webhook_service.verify_signature(site_id, payload, signature):
        raise HTTPException(401, "Invalid signature")
    
    # Parse event
    event = LansweeperWebhookEvent.parse_raw(payload)
    
    # Idempotency check (event ID)
    if await IdempotencyStore.exists(event.event_id):
        return {"status": "duplicate"}
    
    # Store for processing
    await IdempotencyStore.set(event.event_id, "received", ttl=86400)
    
    # Queue for async processing
    await process_lansweeper_event.delay(site_id, event.model_dump())
    
    return {"status": "accepted"}

@celery_app.task
def process_lansweeper_event(site_id: str, event_data: dict):
    event = LansweeperWebhookEvent(**event_data)
    site = LansweeperSiteRepository.get(site_id)
    
    if event.type == "asset.created" or event.type == "asset.updated":
        asset_data = event.payload.asset
        discovered_asset = Normalizer.normalize(asset_data)
        ReconciliationEngine.reconcile(discovered_asset, site.tenant_id)
    
    elif event.type == "asset.deleted":
        # Soft delete in CMDB
        AssetRepository.mark_deleted(event.payload.asset_id, site.tenant_id)
    
    # Record processing
    IdempotencyStore.set(event.event_id, "processed", ttl=86400)
```

### 4.3 Delta Detection (Fallback)

When webhooks are unavailable, incremental sync uses `lastChanged` timestamp:

```graphql
query GetChangedAssets($since: DateTime!, $first: Int!, $after: String) {
  assets(
    first: $first
    after: $after
    filter: { lastChanged: { gte: $since } }
    orderBy: { field: LAST_CHANGED, direction: ASC }
  ) {
    pageInfo { hasNextPage, endCursor }
    edges { cursor, node { ...AssetFields } }
  }
}
```

---

## 5. Reconciliation Engine

### 5.1 Reconciliation Process

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              RECONCILIATION FLOW                                                    │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  Discovered Asset (from Lansweeper)                                                                 │
│           │                                                                                         │
│           ▼                                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  CANDIDATE MATCHING                                                                         │   │
│  │  • Exact match: MAC address, Serial Number, BIOS UUID, Cloud Resource ID                    │   │
│  │  • Fuzzy match: Name (Jaro-Winkler ≥ 0.85), Model + Manufacturer                             │   │
│  │  • Returns: List[Asset] with match_confidence (0.0-1.0)                                     │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│           │                                                                                         │
│           ▼                                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  CONFIDENCE EVALUATION                                                                      │   │
│  │  • ≥ 0.95: AUTO_MERGE (high confidence)                                                     │   │
│  │  • 0.85 - 0.95: MANUAL_REVIEW (queue for steward)                                          │   │
│  │  • < 0.85: NEW_ASSET (create new CI)                                                        │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│           │                                                                                         │
│           ▼                                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  FIELD OWNERSHIP RESOLUTION                                                                 │   │
│  │  For each field: apply Field Ownership Matrix (Section 6)                                   │   │
│  │  • Source priority: LANSWEEPER > CLOUD_API > MANUAL                                        │   │
│  │  • Lock fields prevent overwrite                                                             │   │
│  │  • Merge strategies: SOURCE_WINS, LAST_WRITE_WINS, CONCATENATE, UNION                       │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│           │                                                                                         │
│           ▼                                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  GOLDEN RECORD CREATION/UPDATE                                                              │   │
│  │  • Merge fields per ownership rules                                                         │   │
│  │  • Preserve audit trail (FieldChange events)                                                │   │
│  │  • Update confidence_score                                                                   │   │
│  │  • Emit AssetReconciled event                                                               │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Matching Algorithm

```python
# backend/nexusasset/lansweeper/domain/services/reconciliation_engine.py
class ReconciliationEngine:
    EXACT_MATCH_FIELDS = [
        "mac_address",
        "serial_number", 
        "bios_uuid",
        "cloud_resource_id"
    ]
    
    FUZZY_MATCH_FIELDS = [
        ("name", 0.9),      # Jaro-Winkler threshold
        ("model", 0.85),
    ]
    
    @classmethod
    def find_candidates(cls, discovered: DiscoveredAsset, tenant_id: TenantId) -> List[MatchCandidate]:
        candidates = []
        
        # 1. Exact matches (highest priority)
        for field in cls.EXACT_MATCH_FIELDS:
            value = getattr(discovered, field)
            if value:
                matches = AssetRepository.find_by_field(tenant_id, field, value)
                for asset in matches:
                    candidates.append(MatchCandidate(
                        asset=asset,
                        match_type=MatchType.EXACT,
                        field=field,
                        confidence=1.0
                    ))
        
        # 2. Fuzzy matches
        for field, threshold in cls.FUZZY_MATCH_FIELDS:
            value = getattr(discovered, field)
            if value:
                matches = AssetRepository.fuzzy_search(tenant_id, field, value, threshold)
                for asset, score in matches:
                    candidates.append(MatchCandidate(
                        asset=asset,
                        match_type=MatchType.FUZZY,
                        field=field,
                        confidence=score
                    ))
        
        # 3. Deduplicate candidates (same asset matched on multiple fields)
        return cls._deduplicate_candidates(candidates)
    
    @classmethod
    def reconcile(cls, discovered: DiscoveredAsset, tenant_id: TenantId) -> ReconciliationResult:
        candidates = cls.find_candidates(discovered, tenant_id)
        
        if not candidates:
            return cls._create_new_asset(discovered, tenant_id)
        
        best_match = max(candidates, key=lambda c: c.confidence)
        
        if best_match.confidence >= 0.95:
            return cls._auto_merge(discovered, best_match.asset, tenant_id)
        elif best_match.confidence >= 0.85:
            return cls._queue_for_review(discovered, candidates, tenant_id)
        else:
            return cls._create_new_asset(discovered, tenant_id)
```

---

## 6. Field Ownership Matrix

### 6.1 Matrix Definition

The **Field Ownership Matrix** defines which source system is authoritative for each asset attribute. This prevents data conflicts when multiple sources (Lansweeper, Cloud APIs, Manual entry) provide values for the same field.

| Field Path | Source Priority | Conflict Resolution | Lock | Merge Strategy | Applies To |
|------------|-----------------|---------------------|------|----------------|------------|
| **Hardware Identity** |
| `hardware.cpu.model` | LANSWEEPER, CLOUD_API, MANUAL | SOURCE_WINS | false | — | SERVER, LAPTOP, VM |
| `hardware.cpu.cores` | LANSWEEPER, CLOUD_API, MANUAL | SOURCE_WINS | false | — | SERVER, LAPTOP, VM |
| `hardware.memory.total_gb` | LANSWEEPER, CLOUD_API, MANUAL | SOURCE_WINS | false | — | SERVER, LAPTOP, VM |
| `hardware.storage[]` | LANSWEEPER, CLOUD_API, MANUAL | UNION | false | CONCATENATE | SERVER, LAPTOP, VM |
| `hardware.network_interfaces[]` | LANSWEEPER, CLOUD_API, MANUAL | UNION | false | CONCATENATE | All |
| `identification.serial_number` | LANSWEEPER, MANUAL | SOURCE_WINS | true | — | Physical assets |
| `identification.bios_uuid` | LANSWEEPER | SOURCE_WINS | true | — | Physical/Virtual |
| `identification.mac_address` | LANSWEEPER, CLOUD_API | SOURCE_WINS | true | — | All |
| **Cloud Identity** |
| `cloud.provider` | CLOUD_API, MANUAL | SOURCE_WINS | false | — | CLOUD_RESOURCE |
| `cloud.resource_id` | CLOUD_API | SOURCE_WINS | true | — | CLOUD_RESOURCE |
| `cloud.region` | CLOUD_API, MANUAL | SOURCE_WINS | false | — | CLOUD_RESOURCE |
| `cloud.account_id` | CLOUD_API | SOURCE_WINS | true | — | CLOUD_RESOURCE |
| `cloud.tags` | CLOUD_API, MANUAL | UNION | false | UNION (dict merge) | CLOUD_RESOURCE |
| **Software** |
| `software[].name` | LANSWEEPER, CLOUD_API, MANUAL | SOURCE_WINS | false | — | All |
| `software[].version` | LANSWEEPER, CLOUD_API, MANUAL | SOURCE_WINS | false | — | All |
| `software[].license_key` | MANUAL | SOURCE_WINS | true | — | All |
| **Ownership & Location** |
| `ownership.owner_id` | MANUAL | SOURCE_WINS | true | — | All |
| `ownership.custodian_id` | MANUAL | SOURCE_WINS | true | — | All |
| `location.datacenter` | MANUAL | SOURCE_WINS | true | — | Physical |
| `location.rack` | MANUAL | SOURCE_WINS | true | — | Physical |
| `location.u_position` | MANUAL | SOURCE_WINS | true | — | Physical |
| `location.geo_coordinates` | MANUAL, CLOUD_API | SOURCE_WINS | false | — | All |
| **Financial** |
| `financial.purchase_cost` | MANUAL | SOURCE_WINS | true | — | All |
| `financial.purchase_date` | MANUAL | SOURCE_WINS | true | — | All |
| `financial.depreciation_method` | MANUAL | SOURCE_WINS | true | — | All |
| `financial.warranty_end` | MANUAL, LANSWEEPER | SOURCE_WINS | false | — | All |
| **Lifecycle** |
| `lifecycle.state` | MANUAL, LANSWEEPER (discovered→authorized) | STATE_MACHINE | false | — | All |
| `lifecycle.state_changed_at` | SYSTEM | SYSTEM_WINS | true | — | All |
| **Metadata** |
| `metadata.tags` | MANUAL, LANSWEEPER, CLOUD_API | UNION | false | UNION | All |
| `metadata.custom_fields` | MANUAL | SOURCE_WINS | true | — | All |
| `metadata.sources` | SYSTEM | SYSTEM_WINS | true | ADD_SOURCE | All |
| `metadata.confidence_score` | SYSTEM | SYSTEM_WINS | true | RECALCULATE | All |
| `metadata.last_seen_at` | LANSWEEPER, CLOUD_API | LATEST_WINS | false | — | All |

### 6.2 Matrix Configuration (Per Tenant)

```python
# backend/nexusasset/lansweeper/domain/value_objects/field_ownership.py
@dataclass(frozen=True)
class FieldOwnershipRule:
    field_path: str                    # Dot notation: "hardware.cpu.model"
    source_priority: List[AssetSource] # [LANSWEEPER, CLOUD_API, MANUAL]
    conflict_resolution: ConflictResolution  # SOURCE_WINS, LAST_WRITE_WINS, MERGE, MANUAL_REVIEW
    lock_field: bool                   # Prevent overwrite once manually set
    merge_strategy: Optional[MergeStrategy]  # For MERGE: CONCATENATE, UNION, CUSTOM
    applies_to_types: List[AssetType]  # Empty = all types
    description: str

# Default matrix (can be overridden per tenant)
DEFAULT_FIELD_OWNERSHIP_MATRIX = [
    FieldOwnershipRule(
        field_path="hardware.cpu.model",
        source_priority=[AssetSource.LANSWEEPER, AssetSource.CLOUD_API, AssetSource.MANUAL],
        conflict_resolution=ConflictResolution.SOURCE_WINS,
        lock_field=False,
        description="CPU model from discovery source"
    ),
    FieldOwnershipRule(
        field_path="ownership.owner_id",
        source_priority=[AssetSource.MANUAL],
        conflict_resolution=ConflictResolution.SOURCE_WINS,
        lock_field=True,
        description="Owner only set manually"
    ),
    # ... 50+ rules
]
```

---

## 7. Conflict Resolution

### 7.1 Conflict Detection

```python
# backend/nexusasset/lansweeper/domain/services/conflict_detector.py
class ConflictDetector:
    @classmethod
    def detect_conflicts(
        cls, 
        discovered: DiscoveredAsset, 
        existing: Asset, 
        rules: List[FieldOwnershipRule]
    ) -> List[FieldConflict]:
        conflicts = []
        
        for rule in rules:
            discovered_value = cls._get_nested_value(discovered, rule.field_path)
            existing_value = cls._get_nested_value(existing, rule.field_path)
            
            if discovered_value is None:
                continue
            
            if existing_value is None:
                continue  # No conflict, field empty
            
            if discovered_value == existing_value:
                continue  # Values match
            
            # Check if field is locked (manual override)
            if rule.lock_field and existing.source == AssetSource.MANUAL:
                conflicts.append(FieldConflict(
                    field_path=rule.field_path,
                    existing_value=existing_value,
                    discovered_value=discovered_value,
                    existing_source=existing.source,
                    discovered_source=AssetSource.LANSWEEPER,
                    resolution=ConflictResolution.MANUAL_REVIEW,
                    reason="Field locked by manual entry"
                ))
                continue
            
            # Apply conflict resolution strategy
            resolution = cls._resolve_by_strategy(rule, discovered_value, existing_value)
            conflicts.append(FieldConflict(
                field_path=rule.field_path,
                existing_value=existing_value,
                discovered_value=discovered_value,
                resolution=resolution
            ))
        
        return conflicts
```

### 7.2 Conflict Queue & Resolution UI

```json
// Conflict representation for UI
{
  "conflict_id": "conf_0192f0c1-23a4-7b8c-9d0e-f1a2b3c4d5e6",
  "asset_id": "ast_123456",
  "tenant_id": "tn_abc123",
  "field_path": "hardware.cpu.model",
  "field_label": "CPU Model",
  "existing": {
    "value": "Intel Xeon Gold 6248",
    "source": "MANUAL",
    "updated_at": "2026-10-15T14:30:00Z",
    "updated_by": "usr_001"
  },
  "discovered": {
    "value": "Intel Xeon Gold 6348",
    "source": "LANSWEEPER",
    "discovered_at": "2026-11-09T10:00:00Z"
  },
  "rule": {
    "source_priority": ["LANSWEEPER", "CLOUD_API", "MANUAL"],
    "conflict_resolution": "SOURCE_WINS",
    "lock_field": false
  },
  "suggested_resolution": "ACCEPT_DISCOVERED",
  "status": "PENDING_REVIEW",
  "created_at": "2026-11-09T10:05:00Z"
}
```

### 7.3 Resolution Actions

| Action | Description | Audit Trail |
|--------|-------------|-------------|
| `ACCEPT_DISCOVERED` | Use Lansweeper value | `FieldConflictResolvedEvent` |
| `ACCEPT_EXISTING` | Keep current value | `FieldConflictResolvedEvent` |
| `MERGE_VALUES` | Combine per merge strategy | `FieldConflictResolvedEvent` |
| `MARK_REVIEWED` | No change, acknowledge conflict | `FieldConflictResolvedEvent` |

---

## 8. Software License Reconciliation

### 8.1 License Matching

```python
# backend/nexusasset/lansweeper/domain/services/license_reconciliation.py
class LicenseReconciliationService:
    @classmethod
    def reconcile_software(cls, asset: Asset, tenant_id: TenantId) -> LicenseReconciliationResult:
        discovered_software = asset.installed_software
        entitlements = LicenseEntitlementRepository.get_by_tenant(tenant_id)
        
        results = []
        for sw in discovered_software:
            match = cls._match_entitlement(sw, entitlements)
            
            if match:
                compliance = cls._calculate_compliance(sw, match)
                results.append(SoftwareLicenseResult(
                    software=sw,
                    entitlement=match,
                    status=compliance.status,
                    surplus=compliance.surplus,
                    deficit=compliance.deficit
                ))
            else:
                results.append(SoftwareLicenseResult(
                    software=sw,
                    entitlement=None,
                    status=ComplianceStatus.UNLICENSED,
                    deficit=1
                ))
        
        return LicenseReconciliationResult(
            asset_id=asset.id,
            results=results,
            overall_status=cls._aggregate_status(results)
        )
    
    @classmethod
    def _match_entitlement(cls, software: SoftwareInstallation, entitlements: List[LicenseEntitlement]) -> Optional[LicenseEntitlement]:
        # Normalize names for matching
        sw_name_norm = normalize_software_name(software.name)
        sw_version_norm = normalize_version(software.version)
        
        for ent in entitlements:
            if (ent.product_name_normalized == sw_name_norm and
                ent.version_matches(sw_version_norm) and
                ent.edition_matches(software.edition)):
                return ent
        return None
```

### 8.2 True-Up Automation

```python
# Monthly true-up job
@celery_app.task
def run_monthly_true_up(tenant_id: str):
    assets = AssetRepository.get_with_software(tenant_id)
    all_results = []
    
    for asset in assets:
        result = LicenseReconciliationService.reconcile_software(asset, tenant_id)
        all_results.extend(result.results)
    
    # Generate true-up report
    true_up = TrueUpReport(
        tenant_id=tenant_id,
        period=current_month(),
        unlicensed_count=sum(1 for r in all_results if r.status == UNLICENSED),
        deficit_count=sum(1 for r in all_results if r.deficit > 0),
        surplus_count=sum(1 for r in all_results if r.surplus > 0),
        estimated_cost=calculate_true_up_cost(all_results),
        details=all_results
    )
    
    # Notify stakeholders
    EventPublisher.publish(LicenseTrueUpGeneratedEvent(
        tenant_id=tenant_id,
        report=true_up
    ))
```

---

## 9. Vulnerability Correlation

### 9.1 CVE Correlation Pipeline

```python
# backend/nexusasset/lansweeper/domain/services/vulnerability_correlation.py
class VulnerabilityCorrelationService:
    NVD_API_URL = "https://services.nvd.nist.gov/rest/json/cves/2.0"
    CACHE_TTL = 86400  # 24 hours
    
    @classmethod
    async def correlate_asset(cls, asset: Asset) -> List[VulnerabilityFinding]:
        findings = []
        
        for sw in asset.installed_software:
            cves = await cls._query_nvd(sw.name, sw.version)
            
            for cve in cves:
                risk_score = cls._calculate_risk_score(
                    cve=cve,
                    asset=asset,
                    software=sw
                )
                
                findings.append(VulnerabilityFinding(
                    asset_id=asset.id,
                    cve_id=cve.id,
                    severity=cve.severity,
                    cvss_score=cve.cvss_score,
                    description=cve.description,
                    affected_software=f"{sw.name} {sw.version}",
                    risk_score=risk_score,
                    exploitability=cls._get_exploitability(cve),
                    remediation=cls._get_remediation(cve)
                ))
        
        # Store findings
        for finding in findings:
            VulnerabilityFindingRepository.save(finding)
        
        # Emit events for critical findings
        critical = [f for f in findings if f.risk_score >= 9.0]
        for finding in critical:
            EventPublisher.publish(CriticalVulnerabilityFoundEvent(
                tenant_id=asset.tenant_id,
                finding=finding
            ))
        
        return findings
    
    @classmethod
    async def _query_nvd(cls, product: str, version: str) -> List[NvdCve]:
        # Check cache first
        cache_key = f"nvd:{product}:{version}"
        cached = await Cache.get(cache_key)
        if cached:
            return cached
        
        # Query NVD API
        params = {
            "keywordSearch": f"{product} {version}",
            "resultsPerPage": 50
        }
        response = await HttpClient.get(cls.NVD_API_URL, params=params)
        cves = [NvdCve.from_nvd(item) for item in response.json()["vulnerabilities"]]
        
        await Cache.set(cache_key, cves, ttl=cls.CACHE_TTL)
        return cves
```

### 9.2 Risk Scoring

```python
@classmethod
def _calculate_risk_score(cls, cve: NvdCve, asset: Asset, software: SoftwareInstallation) -> float:
    """Calculate risk score: CVSS × Exploitability × Asset Criticality"""
    
    cvss = cve.cvss_score or 0.0  # 0-10
    exploitability = cls._get_exploitability(cve)  # 0-1
    
    # Asset criticality based on type, tags, location
    criticality = cls._get_asset_criticality(asset)  # 0.5-2.0
    
    # Internet-facing increases risk
    internet_facing = 1.5 if cls._is_internet_facing(asset) else 1.0
    
    # Business critical tag
    business_critical = 1.5 if "business-critical" in asset.tags else 1.0
    
    risk_score = cvss * exploitability * criticality * internet_facing * business_critical
    
    return min(risk_score, 10.0)  # Cap at 10
```

---

## 10. Credential Management

### 10.1 Credential Storage (Vault)

```yaml
# Vault path structure
secret/data/nxa/prod/lansweeper/sites/
├── site_001/
│   ├── auth_type: "pat" | "oauth2"
│   ├── pat_token: "lansweeper_pat_xxx"
│   ├── oauth2_client_id: "nxa-client"
│   ├── oauth2_client_secret: "xxx"
│   ├── oauth2_token_url: "https://lansweeper.example.com/oauth/token"
│   └── graphql_endpoint: "https://lansweeper.example.com/graphql"
├── site_002/
│   └── ...
```

### 10.2 Credential Rotation

```python
# backend/nexusasset/lansweeper/infrastructure/credential_rotation.py
@celery_app.task
def rotate_lansweeper_credentials():
    """Rotate PAT/OAuth credentials per policy (90 days)"""
    sites = LansweeperSiteRepository.get_all_active()
    
    for site in sites:
        if site.auth_config.needs_rotation():
            if site.auth_config.type == "pat":
                new_pat = generate_lansweeper_pat(site)  # Via Lansweeper API
                Vault.update_secret(site.credential_ref, {"pat_token": new_pat})
            elif site.auth_config.type == "oauth2":
                new_secret = rotate_oauth_client_secret(site)
                Vault.update_secret(site.credential_ref, {"oauth2_client_secret": new_secret})
            
            # Test new credentials
            if not test_connectivity(site):
                # Rollback
                Vault.rollback_secret(site.credential_ref)
                alert_oncall(f"Credential rotation failed for site {site.id}")
            else:
                site.auth_config.last_rotated = utc_now()
                LansweeperSiteRepository.update(site)
```

---

## 11. Monitoring & Observability

### 11.1 Key Metrics

| Metric | Type | Description | Alert Threshold |
|--------|------|-------------|-----------------|
| `lansweeper_sync_duration_seconds` | Histogram | Sync job duration | p99 > 3600s |
| `lansweeper_assets_processed_total` | Counter | Assets processed per sync | — |
| `lansweeper_sync_status` | Gauge | 0=success, 1=partial, 2=failed | = 2 |
| `lansweeper_reconciliation_conflicts` | Counter | Conflicts requiring review | > 100/day |
| `lansweeper_webhook_received_total` | Counter | Webhooks received | — |
| `lansweeper_webhook_processing_seconds` | Histogram | Webhook processing time | p99 > 30s |
| `lansweeper_graphql_request_duration_seconds` | Histogram | GraphQL query latency | p99 > 10s |
| `lansweeper_graphql_errors_total` | Counter | GraphQL errors (by type) | > 10/min |
| `lansweeper_credential_test_status` | Gauge | Connectivity test result | = 0 (failure) |
| `lansweeper_license_true_up_findings` | Gauge | Unlicensed/deficit count | > 0 |

### 11.2 Dashboards

- **Sync Overview**: Success rate, throughput, duration trends
- **Reconciliation Health**: Conflict rate, auto-merge rate, queue depth
- **Webhook Health**: Delivery latency, retry rate, failure rate
- **License Compliance**: True-up trends, unlicensed software
- **Vulnerability Posture**: Critical findings, remediation time

---

## 12. Error Handling & Retry

### 12.1 Error Classification

| Error Type | Examples | Retry | Action |
|------------|----------|-------|--------|
| **Transient** | Network timeout, 5xx, rate limit | Yes (exponential) | Auto-retry |
| **Auth Failure** | 401, invalid token, expired PAT | No | Alert, rotation |
| **Validation** | Schema mismatch, missing required fields | No | DLQ, alert |
| **Conflict** | Reconciliation conflict (manual review) | No | Queue for review |
| **Permanent** | 404 (asset deleted), 403 (no access) | No | Mark deleted/skip |

### 12.2 Dead Letter Queue

```python
# Failed sync items go to DLQ for manual inspection
class SyncDeadLetterQueue:
    @classmethod
    def enqueue(cls, job_id: str, asset_external_id: str, error: SyncError):
        dlq_item = SyncDLQItem(
            job_id=job_id,
            asset_external_id=asset_external_id,
            error=error,
            payload=error.raw_payload,
            attempts=error.attempts,
            created_at=utc_now()
        )
        DLQRepository.add(dlq_item)
        EventPublisher.publish(SyncItemDeadLetteredEvent(
            job_id=job_id,
            asset_external_id=asset_external_id,
            error_code=error.code
        ))
```

---

## 13. Security Considerations

| Concern | Mitigation |
|---------|------------|
| **Credential Exposure** | Vault storage, no logging, rotation, least privilege |
| **Data in Transit** | TLS 1.3 for all GraphQL/webhook communication |
| **Webhook Spoofing** | HMAC-SHA256 signature verification, secret rotation |
| **Tenant Isolation** | Sync jobs scoped to tenant_id; RLS in CMDB |
| **Rate Limiting** | Client-side throttling, respect Lansweeper limits |
| **Audit Trail** | All sync decisions logged with correlation IDs |
| **PII Protection** | User names, emails hashed in sync logs |

---

## 14. Diagrams

| Diagram ID | Title | Type | Location | Description |
|------------|-------|------|----------|-------------|
| **DIA-081** | Integration Architecture | Component | Section 1.1 | End-to-end flow |
| **DIA-082** | GraphQL Query Structure | Sequence | Section 2.1 | Paginated asset query |
| **DIA-083** | Sync Orchestrator | Flowchart | Section 3.2 | Celery task flow |
| **DIA-084** | Reconciliation Flow | Flowchart | Section 5.1 | Candidate → Confidence → Ownership → Golden Record |
| **DIA-085** | Field Ownership Matrix | Matrix | Section 6.1 | Field → Source Priority |
| **DIA-086** | Conflict Resolution UI | Component | Section 7.2 | Side-by-side comparison |
| **DIA-087** | Webhook Delivery | Sequence | Section 4.2 | Lansweeper → Traefik → Processor → Reconcile |
| **DIA-088** | License Reconciliation | Flowchart | Section 8 | Discovered SW → Entitlement Match → Compliance |
| **DIA-089** | Vulnerability Correlation | Sequence | Section 9 | Asset SW → NVD → Risk Score → Finding |
| **DIA-090** | Credential Rotation | Sequence | Section 10.2 | Schedule → Generate → Test → Store → Alert |
| **DIA-091** | Sync Monitoring Dashboard | Component | Section 11.2 | Key metrics panels |

---

## 15. Tables

| Table ID | Title | Location | Row Count |
|----------|-------|----------|-----------|
| **TBL-179** | Supported Lansweeper Versions | Section 1.2 | 3 |
| **TBL-180** | Asset Types Synchronized | Section 1.3 | 9 |
| **TBL-181** | GraphQL Client Config | Section 2.2 | 4 |
| **TBL-182** | Rate Limits | Section 2.3 | 4 |
| **TBL-183** | Sync Job Types | Section 3.1 | 4 |
| **TBL-184** | Field Ownership Matrix | Section 6.1 | 30+ |
| **TBL-185** | Conflict Resolution Actions | Section 7.3 | 4 |
| **TBL-186** | Error Classification | Section 12.1 | 5 |
| **TBL-187** | Security Mitigations | Section 13 | 7 |
| **TBL-188** | Key Metrics | Section 11.1 | 10 |
| **TBL-189** | Diagram Index | Section 14 | 11 |
| **TBL-190** | Table Index | Section 15 | 11 |

---

---

**End of DOC-08 — NexusAsset Cloud Lansweeper Integration**

*Document Control: Version 1.0.0 | Classification: Internal — Confidential | Next Review: 2026-12-09*

*For questions or updates: docs@nexusasset.cloud | Integration Team: integrations@nexusasset.cloud*