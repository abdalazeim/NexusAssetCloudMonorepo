# DOC-14: Data Migration & Seed Strategy

---

| **Document ID** | DOC-14 |
|-----------------|--------|
| **Title** | Data Migration & Seed Strategy |
| **Version** | 1.0.0 |
| **Status** | ✓ Approved |
| **Owner** | Data Engineering / Platform |
| **Last Updated** | 2026-11-09 |
| **Classification** | Internal — Confidential |
| **Related Docs** | [DOC-04 Domain Model](../00-overview/04-domain-model.md), [DOC-05 Application Architecture](../00-overview/05-application-architecture.md), [DOC-06 Infrastructure Architecture](../00-overview/06-infrastructure-architecture.md), [DOC-07 API Design](../00-overview/07-api-design.md), [DOC-08 Lansweeper Integration](../00-overview/08-lansweeper-integration.md), [DOC-11 DevOps & CI/CD](../00-overview/11-devops-cicd.md) |
| **Traceability** | REQ-FN-DM-###, REQ-NFR-DM-###, ADR-025..030, TC-DM-### |

---

## 1. Executive Summary

This document defines the **comprehensive data migration and seed strategy** for NexusAsset Cloud, covering: **tenant onboarding migrations**, **schema evolution patterns**, **reference data seeding**, **Lansweeper historical data import**, **multi-environment promotion**, **rollback procedures**, and **zero-downtime deployment guarantees**. The strategy ensures **data integrity**, **tenant isolation**, **audit compliance**, and **<5 min RTO** for migration operations.

> **Note:** All migrations are **idempotent**, **reversible**, and **tenant-scoped**. Zero-downtime achieved via **expand/contract pattern**, **dual-write**, and **feature flags**.

---

## 2. Migration Architecture

### 2.1 Migration Taxonomy

| Type | Scope | Trigger | Frequency | Downtime | Examples |
|------|-------|---------|-----------|----------|----------|
| **Schema Migration** | Global (all tenants) | Deployment | Per release | Zero (expand/contract) | Add column, create index, alter type |
| **Tenant Onboarding** | Single tenant | Provisioning API | On-demand | N/A (async) | New tenant schema, seed data, RLS policies |
| **Data Migration** | Global or Tenant | Feature release | Per release | Zero (dual-write) | Backfill computed fields, normalize data |
| **Reference Data Seed** | Global | Deployment/Manual | Per release + ad-hoc | Zero | License types, vulnerability severities, asset categories |
| **Historical Import** | Tenant | Onboarding/Legacy | One-time + incremental | N/A (async) | Lansweeper full sync, CSV import, API bulk load |
| **Rollback Migration** | Global/Tenant | Incident | On-demand | Zero (if expand) | Revert column, drop index, restore seed |

### 2.2 Migration Execution Environments

| Environment | Purpose | Migration Strategy | Approval |
|-------------|---------|-------------------|----------|
| **Local Dev** | Developer loop | Auto-apply on `manage.py migrate` | None |
| **CI/Test** | PR validation | Fresh DB + full migration stack | Auto (GitHub Actions) |
| **Staging** | Pre-prod validation | Clone prod (anonymized) + migrate | Auto (ArgoCD) |
| **Production** | Live tenants | **Blue/Green + Canary** | **Manual (2-person)** |

---

## 3. Schema Migration Framework (Django + Custom)

### 3.1 Migration File Structure

```
backend/
└── nexusasset/
    └── migrations/
        ├── 0001_initial.py                    # Baseline schema
        ├── 0002_add_asset_foreign_keys.py     # Dependency-ordered
        ├── 0003_tenant_rls_policies.py        # RLS setup
        ├── 0004_seed_license_types.py         # Data migration (RunPython)
        ├── 0005_add_asset_computed_fields.py  # Expand phase
        ├── 0006_backfill_computed_fields.py   # Data backfill
        ├── 0007_remove_legacy_fields.py       # Contract phase
        └── 0008_optimize_indexes.py           # Performance
```

### 3.2 Migration Standards

```python
# backend/nexusasset/core/migrations/0005_add_asset_computed_fields.py
# -*- coding: utf-8 -*-
"""
EXPAND PHASE: Add computed fields for asset search optimization.
- Adds: search_vector (tsvector), risk_score (int), last_seen_days (int)
- Zero-downtime: nullable fields, no default, populated async
- Rollback: 0007_remove_legacy_fields.py (CONTRACT PHASE)
"""
from django.db import migrations, models
import django.contrib.postgres.search as pg_search


class Migration(migrations.Migration):

    dependencies = [
        ('nxa_assets', '0004_seed_license_types'),
    ]

    operations = [
        # EXPAND: Add nullable columns (no table rewrite)
        migrations.AddField(
            model_name='asset',
            name='search_vector',
            field=pg_search.SearchVectorField(null=True, editable=False),
            preserve_default=False,
        ),
        migrations.AddField(
            model_name='asset',
            name='risk_score',
            field=models.PositiveSmallIntegerField(null=True, db_index=True),
            preserve_default=False,
        ),
        migrations.AddField(
            model_name='asset',
            name='last_seen_days',
            field=models.PositiveIntegerField(null=True, db_index=True),
            preserve_default=False,
        ),
        # GIN index for full-text search (CONCURRENTLY for zero-downtime)
        migrations.RunSQL(
            sql="CREATE INDEX CONCURRENTLY IF NOT EXISTS asset_search_vector_gin "
                "ON nxa_assets_asset USING GIN (search_vector);",
            reverse_sql="DROP INDEX CONCURRENTLY IF EXISTS asset_search_vector_gin;",
            state_operations=[],
        ),
        # Partial index for risk scoring
        migrations.RunSQL(
            sql="CREATE INDEX CONCURRENTLY IF NOT EXISTS asset_risk_score_partial "
                "ON nxa_assets_asset (risk_score) WHERE risk_score IS NOT NULL;",
            reverse_sql="DROP INDEX CONCURRENTLY IF EXISTS asset_risk_score_partial;",
            state_operations=[],
        ),
    ]
```

### 3.3 Data Migration Pattern (RunPython)

```python
# backend/nexusasset/nxa_assets/migrations/0006_backfill_computed_fields.py
"""
BACKFILL PHASE: Populate computed fields in batches.
- Idempotent: Uses UPDATE ... WHERE field IS NULL
- Batched: 10,000 rows per transaction
- Progress: Logs every 100k rows
- Resumable: Can be re-run safely
"""

BATCH_SIZE = 10_000
LOG_INTERVAL = 100_000


def backfill_search_vector(apps, schema_editor):
    Asset = apps.get_model('nxa_assets', 'Asset')
    db_alias = schema_editor.connection.alias
    
    # Use raw SQL for performance (bypasses ORM)
    with schema_editor.connection.cursor() as cursor:
        # Get total count for progress
        cursor.execute("SELECT COUNT(*) FROM nxa_assets_asset WHERE search_vector IS NULL")
        total = cursor.fetchone()[0]
        
        processed = 0
        while processed < total:
            cursor.execute(f"""
                UPDATE nxa_assets_asset
                SET search_vector = to_tsvector('english', 
                    COALESCE(name, '') || ' ' || 
                    COALESCE(description, '') || ' ' || 
                    COALESCE(serial_number, '') || ' ' || 
                    COALESCE(mac_addresses, '') || ' ' || 
                    COALESCE(ip_addresses, '')
                )
                WHERE search_vector IS NULL
                AND id IN (
                    SELECT id FROM nxa_assets_asset 
                    WHERE search_vector IS NULL 
                    ORDER BY id 
                    LIMIT {BATCH_SIZE}
                    FOR UPDATE SKIP LOCKED
                )
            """)
            updated = cursor.rowcount
            processed += updated
            
            if processed % LOG_INTERVAL == 0 or updated == 0:
                logger.info(f"Backfill search_vector: {processed}/{total} ({processed/total*100:.1f}%)")
            
            if updated == 0:
                break


def backfill_risk_score(apps, schema_editor):
    Asset = apps.get_model('nxa_assets', 'Asset')
    Vulnerability = apps.get_model('nxa_assets', 'Vulnerability')
    db_alias = schema_editor.connection.alias
    
    with schema_editor.connection.cursor() as cursor:
        # Risk score: sum of CVSS scores for open vulnerabilities
        cursor.execute(f"""
            UPDATE nxa_assets_asset a
            SET risk_score = COALESCE((
                SELECT SUM(v.cvss_score)::int
                FROM nxa_assets_vulnerability v
                WHERE v.asset_id = a.id
                AND v.status = 'open'
            ), 0)
            WHERE risk_score IS NULL
            AND id IN (
                SELECT id FROM nxa_assets_asset 
                WHERE risk_score IS NULL 
                ORDER BY id 
                LIMIT {BATCH_SIZE}
                FOR UPDATE SKIP LOCKED
            )
        """)
        # ... similar logging


def backfill_last_seen_days(apps, schema_editor):
    # ... similar pattern


class Migration(migrations.Migration):

    dependencies = [
        ('nxa_assets', '0005_add_asset_computed_fields'),
    ]

    operations = [
        migrations.RunPython(backfill_search_vector, reverse_code=migrations.RunPython.noop),
        migrations.RunPython(backfill_risk_score, reverse_code=migrations.RunPython.noop),
        migrations.RunPython(backfill_last_seen_days, reverse_code=migrations.RunPython.noop),
    ]
```

### 3.4 Contract Phase (Cleanup)

```python
# backend/nexusasset/nxa_assets/migrations/0007_remove_legacy_fields.py
"""
CONTRACT PHASE: Remove legacy fields after dual-write period.
- Only after: feature flag at 100%, monitoring confirms no reads of old fields
- Rollback: Re-add columns (nullable) + backfill from new fields
"""

class Migration(migrations.Migration):

    dependencies = [
        ('nxa_assets', '0006_backfill_computed_fields'),
    ]

    operations = [
        # Make computed fields NOT NULL (after backfill verified)
        migrations.AlterField(
            model_name='asset',
            name='search_vector',
            field=pg_search.SearchVectorField(null=False, editable=False),
        ),
        migrations.AlterField(
            model_name='asset',
            name='risk_score',
            field=models.PositiveSmallIntegerField(default=0, db_index=True),
        ),
        migrations.AlterField(
            model_name='asset',
            name='last_seen_days',
            field=models.PositiveIntegerField(default=0, db_index=True),
        ),
        # Remove legacy columns (if any)
        # migrations.RemoveField(model_name='asset', name='legacy_field'),
    ]
```

---

## 4. Tenant Onboarding Migration

### 4.1 Onboarding Flow

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        TENANT ONBOARDING MIGRATION FLOW                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  1. PROVISIONING API REQUEST                                                    │
│     POST /api/v1/tenants/ {name, tier, admin_email, region}                    │
│           │                                                                     │
│           ▼                                                                     │
│  2. TENANT RECORD CREATION (nxa_tenants_tenant)                                │
│     • tenant_id = UUIDv7 (time-ordered)                                        │
│     • schema_name = f"tenant_{tenant_id.replace('-', '_')}"                   │
│     • tier, region, status='provisioning'                                      │
│           │                                                                     │
│           ▼                                                                     │
│  3. ASYNC PROVISIONING JOB (Celery: tenant.provision)                          │
│     ┌─────────────────────────────────────────────────────────────────────┐    │
│     │ a) CREATE SCHEMA (PostgreSQL)                                        │    │
│     │    CREATE SCHEMA tenant_abc123;                                      │    │
│     │    GRANT USAGE ON SCHEMA tenant_abc123 TO nxa_app;                  │    │
│     │                                                                      │    │
│     │ b) RUN MIGRATIONS IN TENANT SCHEMA                                   │    │
│     │    ./manage.py migrate --schema=tenant_abc123                        │    │
│     │                                                                      │    │
│     │ c) APPLY RLS POLICIES                                                │    │
│     │    ALTER TABLE tenant_abc123.assets ENABLE ROW LEVEL SECURITY;      │    │
│     │    CREATE POLICY tenant_isolation ON tenant_abc123.assets            │    │
│     │      USING (tenant_id = current_setting('app.current_tenant')::uuid);│    │
│     │                                                                      │    │
│     │ d) SEED REFERENCE DATA (tenant-scoped)                               │    │
│     │    • Default asset categories, statuses, custom fields               │    │
│     │    • Default notification templates                                  │    │
│     │    • Default sync schedules                                          │    │
│     │                                                                      │    │
│     │ e) CREATE ADMIN USER                                                 │    │
│     │    • Invite sent to admin_email                                      │    │
│     │    • Role: tenant_admin                                              │    │
│     │                                                                      │    │
│     │ f) INITIALIZE BILLING                                                │    │
│     │    • Create subscription (tier-based)                                │    │
│     │    • Set up usage quotas                                             │    │
│     └─────────────────────────────────────────────────────────────────────┘    │
│           │                                                                     │
│           ▼                                                                     │
│  4. STATUS UPDATE: tenant.status = 'active'                                    │
│           │                                                                     │
│           ▼                                                                     │
│  5. WEBHOOK: tenant.provisioned → BFF, Billing, Notifications                  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Tenant Provisioning Job

```python
# backend/nexusasset/nxa_tenants/tasks/provisioning.py
from celery import shared_task
from django.db import connection, transaction
from django.core.management import call_command
import logging

logger = logging.getLogger(__name__)


@shared_task(
    bind=True,
    max_retries=3,
    default_retry_delay=60,
    autoretry_for=(Exception,),
    retry_backoff=True,
    retry_backoff_max=300,
)
def provision_tenant(self, tenant_id: str):
    """
    Provision a new tenant schema and seed data.
    Idempotent: Can be safely retried.
    """
    from nxa_tenants.models import Tenant
    
    tenant = Tenant.objects.get(id=tenant_id)
    schema_name = tenant.schema_name
    
    try:
        with transaction.atomic():
            # 1. Create schema
            with connection.cursor() as cursor:
                cursor.execute(f'CREATE SCHEMA IF NOT EXISTS "{schema_name}"')
                cursor.execute(
                    f'GRANT USAGE ON SCHEMA "{schema_name}" TO nxa_app'
                )
                cursor.execute(
                    f'ALTER DEFAULT PRIVILEGES IN SCHEMA "{schema_name}" '
                    f'GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO nxa_app'
                )
        
        # 2. Run migrations in tenant schema
        # Uses Django's schema_editor with search_path
        from django.db.backends.postgresql.schema import DatabaseSchemaEditor
        from django.db import connections
        
        conn = connections['default']
        with conn.cursor() as cursor:
            cursor.execute(f'SET search_path TO "{schema_name}", public')
        
        call_command('migrate', '--database=default', '--run-syncdb', verbosity=1)
        
        # 3. Apply RLS policies (idempotent)
        _apply_rls_policies(schema_name)
        
        # 4. Seed tenant reference data
        _seed_tenant_reference_data(schema_name)
        
        # 5. Create admin user invitation
        _create_admin_invitation(tenant)
        
        # 6. Initialize billing
        _initialize_billing(tenant)
        
        # 7. Update status
        tenant.status = 'active'
        tenant.provisioned_at = timezone.now()
        tenant.save(update_fields=['status', 'provisioned_at'])
        
        logger.info(f"Tenant {tenant_id} provisioned successfully")
        
        # 8. Emit event
        from nxa_core.events import emit_event
        emit_event('tenant.provisioned', {
            'tenant_id': str(tenant.id),
            'schema_name': schema_name,
            'tier': tenant.tier,
            'region': tenant.region,
        })
        
    except Exception as exc:
        tenant.status = 'failed'
        tenant.provision_error = str(exc)
        tenant.save(update_fields=['status', 'provision_error'])
        logger.error(f"Tenant {tenant_id} provisioning failed: {exc}")
        raise


def _apply_rls_policies(schema_name: str):
    """Apply Row Level Security policies to all tenant tables."""
    from django.db import connection
    
    policies = [
        # Core tables
        ('nxa_core_user', 'tenant_id'),
        ('nxa_core_apitoken', 'tenant_id'),
        # Asset tables
        ('nxa_assets_asset', 'tenant_id'),
        ('nxa_assets_vulnerability', 'asset__tenant_id'),  # Join
        ('nxa_assets_softwarelicense', 'tenant_id'),
        # Sync tables
        ('nxa_lansweeper_syncjob', 'tenant_id'),
        ('nxa_lansweeper_assetraw', 'tenant_id'),
        # Billing
        ('nxa_billing_subscription', 'tenant_id'),
        ('nxa_billing_invoice', 'tenant_id'),
        # Notifications
        ('nxa_notifications_notification', 'tenant_id'),
        ('nxa_notifications_template', 'tenant_id'),
        # Audit
        ('nxa_audit_auditlog', 'tenant_id'),
    ]
    
    with connection.cursor() as cursor:
        for table, tenant_column in policies:
            # Enable RLS
            cursor.execute(f'ALTER TABLE "{schema_name}"."{table}" ENABLE ROW LEVEL SECURITY')
            
            # Create policy (idempotent)
            policy_name = f"{table}_tenant_isolation"
            cursor.execute(f"""
                DO $$
                BEGIN
                    IF NOT EXISTS (
                        SELECT 1 FROM pg_policies 
                        WHERE schemaname = '{schema_name}' 
                        AND tablename = '{table}' 
                        AND policyname = '{policy_name}'
                    ) THEN
                        CREATE POLICY {policy_name} ON "{schema_name}"."{table}"
                        USING ({tenant_column} = current_setting('app.current_tenant')::uuid);
                    END IF;
                END $$;
            """)


def _seed_tenant_reference_data(schema_name: str):
    """Seed tenant-specific reference data."""
    from django.db import connection
    
    seed_data = {
        'nxa_assets_assetcategory': [
            {'name': 'Hardware', 'description': 'Physical devices', 'sort_order': 10},
            {'name': 'Software', 'description': 'Applications and licenses', 'sort_order': 20},
            {'name': 'Virtual', 'description': 'VMs, containers, cloud resources', 'sort_order': 30},
            {'name': 'Network', 'description': 'Switches, routers, firewalls', 'sort_order': 40},
            {'name': 'Peripheral', 'description': 'Monitors, printers, accessories', 'sort_order': 50},
        ],
        'nxa_assets_assetstatus': [
            {'name': 'Active', 'description': 'In use', 'color': '#10B981', 'sort_order': 10},
            {'name': 'Inactive', 'description': 'Not in use', 'color': '#6B7280', 'sort_order': 20},
            {'name': 'Maintenance', 'description': 'Under repair', 'color': '#F59E0B', 'sort_order': 30},
            {'name': 'Retired', 'description': 'End of life', 'color': '#EF4444', 'sort_order': 40},
            {'name': 'Lost', 'description': 'Missing/stolen', 'color': '#DC2626', 'sort_order': 50},
        ],
        'nxa_assets_customfield': [
            {'name': 'Cost Center', 'field_type': 'text', 'is_required': False},
            {'name': 'Owner', 'field_type': 'user', 'is_required': False},
            {'name': 'Location', 'field_type': 'text', 'is_required': False},
            {'name': 'Warranty Expiry', 'field_type': 'date', 'is_required': False},
        ],
    }
    
    with connection.cursor() as cursor:
        cursor.execute(f'SET search_path TO "{schema_name}", public')
        
        for table, rows in seed_data.items():
            columns = list(rows[0].keys())
            placeholders = ', '.join(['%s'] * len(columns))
            column_names = ', '.join(f'"{c}"' for c in columns)
            
            # Upsert (idempotent)
            for row in rows:
                values = [row[c] for c in columns]
                cursor.execute(f"""
                    INSERT INTO "{schema_name}"."{table}" ({column_names})
                    VALUES ({placeholders})
                    ON CONFLICT DO NOTHING
                """, values)
```

### 4.3 Tenant Deprovisioning (GDPR Compliance)

```python
# backend/nexusasset/nxa_tenants/tasks/deprovisioning.py

@shared_task(bind=True, max_retries=2)
def deprovision_tenant(self, tenant_id: str, confirm: bool = False):
    """
    Deprovision tenant (GDPR Article 17 - Right to Erasure).
    Requires explicit confirmation.
    """
    if not confirm:
        raise ValueError("Deprovisioning requires confirm=True")
    
    from nxa_tenants.models import Tenant
    tenant = Tenant.objects.get(id=tenant_id)
    schema_name = tenant.schema_name
    
    # 1. Anonymize audit logs (retain for compliance, remove PII)
    _anonymize_audit_logs(schema_name)
    
    # 2. Drop tenant schema (cascades to all tables)
    with connection.cursor() as cursor:
        cursor.execute(f'DROP SCHEMA IF EXISTS "{schema_name}" CASCADE')
    
    # 3. Delete tenant record
    tenant.delete()
    
    # 4. Emit event
    emit_event('tenant.deprovisioned', {'tenant_id': tenant_id})
    
    logger.info(f"Tenant {tenant_id} deprovisioned (GDPR erasure)")
```

---

## 5. Reference Data Seeding Strategy

### 5.1 Seed Data Categories

| Category | Source | Scope | Update Frequency | Examples |
|----------|--------|-------|------------------|----------|
| **System Reference** | Code (migrations) | Global | Per release | License types, vuln severities, asset categories |
| **Tenant Defaults** | Code (migrations) | Per tenant | Onboarding + releases | Default statuses, custom fields, notification templates |
| **Configuration** | Database (admin UI) | Global/Tenant | Ad-hoc | Feature flags, integration configs, pricing tiers |
| **External Sync** | APIs (Lansweeper, NVD) | Per tenant | Scheduled (daily) | Software products, vulnerabilities, CVEs |

### 5.2 Global Seed Migrations (Run Once)

```python
# backend/nexusasset/nxa_core/migrations/0010_seed_global_reference_data.py
"""
GLOBAL REFERENCE DATA: Seeded once at install, updated via migrations.
Immutable IDs (UUIDv5) for cross-environment consistency.
"""

GLOBAL_SEEDS = {
    'nxa_core_licensetype': [
        # (id, name, description, metric, compliance_frameworks)
        ('550e8400-e29b-41d4-a716-446655440001', 'Per Device', 'Licensed per physical device', 'device', ['SAM']),
        ('550e8400-e29b-41d4-a716-446655440002', 'Per User', 'Licensed per named user', 'user', ['SAM']),
        ('550e8400-e29b-41d4-a716-446655440003', 'Per Core', 'Licensed per CPU core', 'core', ['SAM', 'Microsoft']),
        ('550e8400-e29b-41d4-a716-446655440004', 'Subscription', 'Recurring subscription', 'subscription', ['SaaS']),
        ('550e8400-e29b-41d4-a716-446655440005', 'Concurrent', 'Licensed per concurrent user', 'concurrent', ['SAM']),
        ('550e8400-e29b-41d4-a716-446655440006', 'Site', 'Licensed per physical location', 'site', ['SAM']),
        ('550e8400-e29b-41d4-a716-446655440007', 'Enterprise', 'Unlimited within organization', 'enterprise', ['EA']),
        ('550e8400-e29b-41d4-a716-446655440008', 'Freeware', 'Free to use', 'freeware', []),
        ('550e8400-e29b-41d4-a716-446655440009', 'Open Source', 'OSI-approved license', 'opensource', ['Compliance']),
        ('550e8400-e29b-41d4-a716-446655440010', 'Custom', 'Custom licensing terms', 'custom', []),
    ],
    'nxa_core_vulnerabilityseverity': [
        ('660e8400-e29b-41d4-a716-446655440001', 'Critical', 9.0, 10.0, '#DC2626', 0),  # CVSS 9-10
        ('660e8400-e29b-41d4-a716-446655440002', 'High', 7.0, 8.9, '#EF4444', 1),      # CVSS 7-8.9
        ('660e8400-e29b-41d4-a716-446655440003', 'Medium', 4.0, 6.9, '#F59E0B', 2),    # CVSS 4-6.9
        ('660e8400-e29b-41d4-a716-446655440004', 'Low', 0.1, 3.9, '#3B82F6', 3),       # CVSS 0.1-3.9
        ('660e8400-e29b-41d4-a716-446655440005', 'None', 0.0, 0.0, '#6B7280', 4),       # CVSS 0
        ('660e8400-e29b-41d4-a716-446655440006', 'Unknown', None, None, '#9CA3AF', 5),   # Not scored
    ],
    'nxa_assets_assettype': [
        ('770e8400-e29b-41d4-a716-446655440001', 'Workstation', 'Desktop/laptop computers', 'hardware'),
        ('770e8400-e29b-41d4-a716-446655440002', 'Server', 'Physical/virtual servers', 'hardware'),
        ('770e8400-e29b-41d4-a716-446655440003', 'Mobile', 'Phones, tablets', 'hardware'),
        ('770e8400-e29b-41d4-a716-446655440004', 'Network Device', 'Switches, routers, APs', 'hardware'),
        ('770e8400-e29b-41d4-a716-446655440005', 'Printer', 'Printers, MFPs', 'hardware'),
        ('770e8400-e29b-41d4-a716-446655440006', 'Storage', 'NAS, SAN, backup appliances', 'hardware'),
        ('770e8400-e29b-41d4-a716-446655440007', 'Virtual Machine', 'VMware, Hyper-V, KVM', 'virtual'),
        ('770e8400-e29b-41d4-a716-446655440008', 'Container', 'Docker, Kubernetes pods', 'virtual'),
        ('770e8400-e29b-41d4-a716-446655440009', 'Cloud Instance', 'AWS EC2, Azure VM, GCP', 'virtual'),
        ('770e8400-e29b-41d4-a716-446655440010', 'Software', 'Installed applications', 'software'),
        ('770e8400-e29b-41d4-a716-446655440011', 'SaaS Subscription', 'Cloud software subscriptions', 'software'),
        ('770e8400-e29b-41d4-a716-446655440012', 'License', 'Software license entitlement', 'software'),
    ],
}


def seed_global_data(apps, schema_editor):
    from django.db import connection
    
    for table, rows in GLOBAL_SEEDS.items():
        # Get columns from model
        model = apps.get_model(*table.split('_', 1))
        columns = [f.name for f in model._meta.fields if f.name != 'id']
        
        with connection.cursor() as cursor:
            for row in rows:
                # row[0] is the deterministic UUID
                values = list(row)
                placeholders = ', '.join(['%s'] * len(values))
                col_names = ', '.join(['id'] + [f'"{c}"' for c in columns])
                
                cursor.execute(f"""
                    INSERT INTO {table} ({col_names})
                    VALUES ({placeholders})
                    ON CONFLICT (id) DO UPDATE SET
                        {', '.join(f'"{c}" = EXCLUDED."{c}"' for c in columns)}
                """, values)


class Migration(migrations.Migration):
    dependencies = [('nxa_core', '0009_add_global_reference_tables')]
    operations = [migrations.RunPython(seed_global_data, migrations.RunPython.noop)]
```

### 5.3 Seed Data Versioning & Updates

```yaml
# backend/nexusasset/nxa_core/seed_data/versions.yaml
# Managed by Data Engineering, applied via migration

seed_versions:
  nxa_core_licensetype:
    current_version: 3
    history:
      - version: 1
        date: "2025-01-15"
        description: "Initial license types"
        migration: "0010_seed_global_reference_data"
      - version: 2
        date: "2025-06-20"
        description: "Added Concurrent, Site, Enterprise types"
        migration: "0015_add_enterprise_license_types"
      - version: 3
        date: "2026-03-10"
        description: "Added Freeware, Open Source, Custom; added compliance_frameworks"
        migration: "0022_update_license_compliance"

  nxa_core_vulnerabilityseverity:
    current_version: 2
    history:
      - version: 1
        date: "2025-01-15"
        description: "CVSS v3.1 severity bands"
      - version: 2
        date: "2025-11-01"
        description: "Added Unknown severity for unscored CVEs"
        migration: "0031_add_unknown_severity"
```

---

## 6. Lansweeper Historical Data Migration

### 6.1 Import Strategy

| Phase | Scope | Method | Duration | Validation |
|-------|-------|--------|----------|------------|
| **Discovery** | All assets | Lansweeper GraphQL API (paginated) | 2-24h | Count match |
| **Full Sync** | All assets + software + users | Parallel workers (10) | 4-48h | Checksum per table |
| **Incremental** | Changed since last sync | Webhook + polling (15min) | Continuous | Lag < 5min |
| **Delta Reconciliation** | Drift detection | Nightly compare | 1-2h | 99.9% match |

### 6.2 Import Pipeline

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      LANSWEEPER HISTORICAL IMPORT PIPELINE                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  LANSWEEPER (Source)                                                            │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  GraphQL API: assets, software, users, monitors, licenses, vulnerabilities│   │
│  │  Rate Limit: 100 req/s | Pagination: 1000/page | Webhook: asset.changed │   │
│  └────────────────────────────────┬────────────────────────────────────────┘   │
│                                   │                                             │
│                                   ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                    LANSWEEPER SYNC WORKER (nxa_lansweeper)              │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │   │
│  │  │  EXTRACT    │→ │  TRANSFORM  │→ │  VALIDATE   │→ │   LOAD      │    │   │
│  │  │             │  │             │  │             │  │             │    │   │
│  │  │ • Paginate  │  │ • Map fields│  │ • Schema    │  │ • Upsert    │    │   │
│  │  │ • Rate limit│  │ • Normalize │  │   validation│  │   (ON CON-  │    │   │
│  │  │ • Checkpoint│  │ • Enrich    │  │ • Business  │  │   FLICT)    │    │   │
│  │  │ • Resume    │  │   (CMDB)    │  │   rules     │  │ • Batch     │    │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │   5000 rows │    │   │
│  │                                                   └─────────────┘    │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                   │                                             │
│               ┌───────────────────┼───────────────────┐                        │
│               ▼                   ▼                   ▼                        │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐             │
│  │  RAW STAGING     │  │  RECONCILED      │  │  MASTER (nxa_    │             │
│  │  (nxa_lansweeper │  │  (nxa_lansweeper │  │  assets_asset)  │             │
│  │   _assetraw)     │  │   _assetreconciled)│  │                  │             │
│  │  • Immutable     │  │  • Merged        │  │  • Golden record │             │
│  │  • Source truth  │  │  • Conflict log  │  │  • Field owners  │             │
│  │  • Partitioned   │  │  • Lineage       │  │  • RLS           │             │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 6.3 Field Ownership & Conflict Resolution

```python
# backend/nexusasset/nxa_lansweeper/services/reconciliation.py
"""
FIELD OWNERSHIP MATRIX: Defines which system owns each field.
Used by reconciliation engine to resolve conflicts.
"""

FIELD_OWNERSHIP = {
    # Asset Core
    'name': 'lansweeper',           # Lansweeper is authoritative
    'asset_type': 'lansweeper',
    'serial_number': 'lansweeper',
    'mac_addresses': 'lansweeper',
    'ip_addresses': 'lansweeper',
    'manufacturer': 'lansweeper',
    'model': 'lansweeper',
    'chassis_type': 'lansweeper',
    
    # Business Context (NexusAsset owned)
    'asset_category': 'nexusasset',      # User categorization
    'asset_status': 'nexusasset',        # Lifecycle management
    'owner': 'nexusasset',               # Assigned user
    'cost_center': 'nexusasset',         # Financial tracking
    'location': 'nexusasset',            # Physical/virtual location
    'custom_fields': 'nexusasset',       # Extensible attributes
    'purchase_date': 'nexusasset',       # Procurement data
    'warranty_expiry': 'nexusasset',     # Contract data
    'notes': 'nexusasset',               # Operational notes
    
    # Software (Shared)
    'software_name': 'lansweeper',       # Discovered
    'software_version': 'lansweeper',    # Discovered
    'software_publisher': 'lansweeper',  # Discovered
    'license_type': 'nexusasset',        # Entitlement
    'license_entitlement': 'nexusasset', # Purchased count
    'install_date': 'lansweeper',        # Discovered
    
    # Vulnerability (NexusAsset enriched)
    'vulnerabilities': 'nexusasset',     # Correlated from NVD
    'risk_score': 'nexusasset',          # Calculated
}


CONFLICT_RESOLUTION_RULES = {
    'lansweeper_wins': [
        'name', 'serial_number', 'mac_addresses', 'ip_addresses',
        'manufacturer', 'model', 'chassis_type', 'asset_type',
        'software_name', 'software_version', 'software_publisher',
        'install_date',
    ],
    'nexusasset_wins': [
        'asset_category', 'asset_status', 'owner', 'cost_center',
        'location', 'custom_fields', 'purchase_date', 'warranty_expiry',
        'notes', 'license_type', 'license_entitlement',
    ],
    'merge_arrays': [
        'mac_addresses', 'ip_addresses', 'tags',
    ],
    'max_value': [
        'risk_score',  # Take highest risk
    ],
    'manual_review': [
        # Fields requiring human intervention
        # Trigger: webhook to notification service
    ],
}


def reconcile_asset(raw_asset: dict, master_asset: Asset | None) -> Asset:
    """
    Reconcile raw Lansweeper data with master asset.
    Returns updated master asset (not saved).
    """
    if master_asset is None:
        # New asset - create from Lansweeper
        return _create_from_lansweeper(raw_asset)
    
    # Existing asset - apply field ownership
    for field, owner in FIELD_OWNERSHIP.items():
        raw_value = raw_asset.get(field)
        master_value = getattr(master_asset, field, None)
        
        if raw_value is None:
            continue
            
        if owner == 'lansweeper':
            # Lansweeper authoritative - overwrite
            setattr(master_asset, field, raw_value)
        elif owner == 'nexusasset':
            # NexusAsset authoritative - preserve master
            pass
        elif owner == 'merge_arrays':
            # Merge unique values
            merged = list(set((master_value or []) + (raw_value or [])))
            setattr(master_asset, field, merged)
        elif owner == 'max_value':
            # Take maximum
            setattr(master_asset, field, max(master_value or 0, raw_value or 0))
    
    # Track reconciliation metadata
    master_asset.last_reconciled_at = timezone.now()
    master_asset.reconciliation_version += 1
    master_asset.lansweeper_last_seen_at = raw_asset.get('last_seen')
    
    return master_asset
```

### 6.4 Import Job Orchestration

```python
# backend/nexusasset/nxa_lansweeper/tasks/import_tasks.py

@shared_task(bind=True, max_retries=3)
def full_historical_import(self, tenant_id: str, sync_job_id: str):
    """
    Full historical import from Lansweeper.
    Runs in chunks with checkpointing for resumability.
    """
    from nxa_lansweeper.models import SyncJob, AssetRaw
    from nxa_lansweeper.services.graphql_client import LansweeperGraphQLClient
    
    sync_job = SyncJob.objects.get(id=sync_job_id)
    tenant = Tenant.objects.get(id=tenant_id)
    client = LansweeperGraphQLClient(tenant.lansweeper_config)
    
    sync_job.status = 'running'
    sync_job.started_at = timezone.now()
    sync_job.save()
    
    try:
        # 1. Fetch total count for progress
        total_assets = client.get_asset_count()
        sync_job.total_records = total_assets
        sync_job.save(update_fields=['total_records'])
        
        # 2. Process in pages
        page_size = 1000
        processed = 0
        cursor = sync_job.checkpoint_cursor  # Resume from last checkpoint
        
        while True:
            # Fetch page
            assets_page, next_cursor = client.fetch_assets(
                first=page_size,
                after=cursor
            )
            
            if not assets_page:
                break
            
            # 3. Bulk upsert to raw staging
            _bulk_upsert_raw_assets(tenant.schema_name, assets_page)
            
            processed += len(assets_page)
            cursor = next_cursor
            
            # 4. Update checkpoint every 10k records
            if processed % 10000 == 0:
                sync_job.processed_records = processed
                sync_job.checkpoint_cursor = cursor
                sync_job.save(update_fields=['processed_records', 'checkpoint_cursor'])
                logger.info(f"Import progress: {processed}/{total_assets}")
            
            if not next_cursor:
                break
        
        # 5. Trigger reconciliation
        sync_job.status = 'reconciling'
        sync_job.save()
        reconcile_tenant_assets.delay(tenant_id, sync_job_id)
        
    except Exception as exc:
        sync_job.status = 'failed'
        sync_job.error_message = str(exc)
        sync_job.save()
        raise


def _bulk_upsert_raw_assets(schema_name: str, assets: list[dict]):
    """Bulk upsert raw assets to staging table."""
    from django.db import connection
    
    # Prepare columns (match AssetRaw model)
    columns = [
        'lansweeper_id', 'asset_id', 'name', 'asset_type', 'serial_number',
        'mac_addresses', 'ip_addresses', 'manufacturer', 'model',
        'chassis_type', 'domain', 'username', 'last_seen', 'raw_data'
    ]
    
    with connection.cursor() as cursor:
        cursor.execute(f'SET search_path TO "{schema_name}", public')
        
        # Use COPY for performance (PostgreSQL)
        # Or execute_batch for smaller batches
        from psycopg2.extras import execute_batch
        
        data = [
            (
                a['id'], a.get('assetId'), a.get('name'), a.get('type'),
                a.get('serialNumber'), a.get('macAddresses', []),
                a.get('ipAddresses', []), a.get('manufacturer'), a.get('model'),
                a.get('chassisType'), a.get('domain'), a.get('username'),
                a.get('lastSeen'), json.dumps(a)
            )
            for a in assets
        ]
        
        execute_batch(cursor, f"""
            INSERT INTO nxa_lansweeper_assetraw ({', '.join(columns)})
            VALUES ({', '.join(['%s'] * len(columns))})
            ON CONFLICT (lansweeper_id) DO UPDATE SET
                {', '.join(f'{c} = EXCLUDED.{c}' for c in columns if c != 'lansweeper_id')}
        """, data, page_size=5000)
```

---

## 7. Multi-Environment Promotion

### 7.1 Migration Promotion Pipeline

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                     MIGRATION PROMOTION PIPELINE                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  DEVELOPER                        CI/TEST                    STAGING            │
│  ─────────                        ───────                    ───────            │
│                                                                                 │
│  1. Create migration           4. Auto-run on          7. ArgoCD deploys       │
│     file locally               PR merge                  to staging             │
│                                                                                 │
│  2. ./manage.py                5. Fresh DB +            8. Run against          │
│     makemigrations             full migration            anonymized prod        │
│     --check                    stack                     clone                  │
│                                                                                 │
│  3. Test locally               6. Validate:             9. Automated tests:     │
│     • migrate                  • Zero errors            • Schema diff           │
│     • test data                • Performance            • Data integrity        │
│     • rollback                 • Rollback works         • RLS policies          │
│                                • Seed data              • Seed completeness     │
│                                                                                 │
│                                    │                                           │
│                                    ▼                                           │
│                         ┌─────────────────────────┐                            │
│                         │      PRODUCTION         │                            │
│                         │      (Blue/Green)       │                            │
│                         │  10. Manual approval    │                            │
│                         │  11. Blue deploy        │                            │
│                         │  12. Canary 5% → 25%    │                            │
│                         │  13. Full cutover       │                            │
│                         │  14. Green cleanup      │                            │
│                         └─────────────────────────┘                            │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Production Deployment Procedure

```yaml
# .github/workflows/migration-production.yml
name: Production Migration Deploy

on:
  workflow_dispatch:
    inputs:
      migration_tag:
        description: 'Git tag of migration release (e.g., migrate-v1.4.0)'
        required: true
      confirm:
        description: 'Type "DEPLOY TO PRODUCTION" to confirm'
        required: true

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.inputs.migration_tag }}
      
      - name: Validate migration files
        run: |
          # Check for dangerous operations
          python scripts/validate_migrations.py --strict
      
      - name: Dry-run on staging clone
        run: |
          ./scripts/dry_run_migration.sh staging-clone-${{ github.run_id }}

  deploy-blue:
    needs: validate
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Blue environment
        run: |
          kubectl set image deployment/nxa-api-blue \
            nxa-api=nexusasset/api:${{ github.event.inputs.migration_tag }} \
            -n nxa-prod-blue
      
      - name: Run migrations on Blue
        run: |
          kubectl exec -n nxa-prod-blue deploy/nxa-api-blue -- \
            ./manage.py migrate --noinput --verbosity=2
      
      - name: Validate Blue health
        run: |
          ./scripts/health_check.sh blue

  canary:
    needs: deploy-blue
    runs-on: ubuntu-latest
    steps:
      - name: Route 5% traffic to Blue
        run: |
          ./scripts/update_traffic_split.sh blue 5 green 95
      
      - name: Monitor canary (10 min)
        run: |
          ./scripts/monitor_canary.sh --duration=600 --error-threshold=0.1%

  promote:
    needs: canary
    runs-on: ubuntu-latest
    steps:
      - name: Progressive traffic shift
        run: |
          for split in 25 50 75 100; do
            ./scripts/update_traffic_split.sh blue $split green $((100-split))
            ./scripts/monitor_canary.sh --duration=300 --error-threshold=0.1%
          done

  cleanup:
    needs: promote
    runs-on: ubuntu-latest
    steps:
      - name: Decommission Green
        run: |
          kubectl scale deployment/nxa-api-green --replicas=0 -n nxa-prod-green
```

---

## 8. Rollback Procedures

### 8.1 Rollback Decision Matrix

| Scenario | Rollback Type | RTO | Procedure |
|----------|---------------|-----|-----------|
| **Schema expand phase** | Forward-only (no rollback needed) | N/A | New columns nullable, no default |
| **Data migration (backfill)** | Re-run with fix | <30 min | Idempotent, fix code, re-run |
| **Contract phase (drop column)** | **Full rollback required** | <5 min | Restore column from backup, re-expand |
| **Seed data corruption** | Truncate + reseed | <10 min | `TRUNCATE ... CASCADE; re-run seed migration` |
| **Tenant provisioning failure** | Delete schema + retry | <2 min | `DROP SCHEMA ... CASCADE; re-provision` |
| **Lansweeper import failure** | Resume from checkpoint | <5 min | Fix issue, re-run from last cursor |

### 8.2 Schema Rollback (Contract Phase)

```python
# backend/nexusasset/nxa_assets/migrations/0007_remove_legacy_fields.py
# REVERSE OPERATIONS for rollback

def reverse_contract_phase(apps, schema_editor):
    """
    Rollback contract phase: Re-add dropped columns, restore data.
    """
    from django.db import connection
    
    with connection.cursor() as cursor:
        # 1. Re-add legacy columns (nullable)
        cursor.execute("""
            ALTER TABLE nxa_assets_asset 
            ADD COLUMN IF NOT EXISTS legacy_field VARCHAR(255)
        """)
        
        # 2. Restore data from new fields (if reversible)
        cursor.execute("""
            UPDATE nxa_assets_asset
            SET legacy_field = new_field::varchar
            WHERE legacy_field IS NULL
        """)
        
        # 3. Drop new indexes (if created in this migration)
        cursor.execute("DROP INDEX CONCURRENTLY IF EXISTS asset_risk_score_partial")
        cursor.execute("DROP INDEX CONCURRENTLY IF EXISTS asset_search_vector_gin")
        
        # 4. Make new fields nullable again
        cursor.execute("""
            ALTER TABLE nxa_assets_asset 
            ALTER COLUMN risk_score DROP NOT NULL,
            ALTER COLUMN search_vector DROP NOT NULL,
            ALTER COLUMN last_seen_days DROP NOT NULL
        """)


class Migration(migrations.Migration):
    dependencies = [('nxa_assets', '0006_backfill_computed_fields')]
    operations = [
        migrations.RunPython(forward_contract, reverse_contract_phase),
    ]
```

### 8.3 Automated Rollback Trigger

```yaml
# .github/workflows/auto-rollback.yml
name: Auto Rollback on Migration Failure

on:
  workflow_run:
    workflows: ["Production Migration Deploy"]
    types: [completed]

jobs:
  check-and-rollback:
    if: ${{ failure() && github.event.workflow_run.conclusion == 'failure' }}
    runs-on: ubuntu-latest
    steps:
      - name: Identify failed stage
        id: failed
        run: |
          # Parse workflow logs to find failed job
          echo "failed_job=canary" >> $GITHUB_OUTPUT
      
      - name: Execute rollback
        run: |
          ./scripts/emergency_rollback.sh \
            --target=green \
            --reason="Auto-rollback: ${{ steps.failed.outputs.failed_job }} failed"
      
      - name: Notify on-call
        run: |
          curl -X POST ${{ secrets.PAGERDUTY_WEBHOOK }} \
            -d '{"summary":"AUTO ROLLBACK EXECUTED","severity":"critical"}'
```

---

## 9. Testing Strategy for Migrations

### 9.1 Migration Test Pyramid

| Level | Scope | Tools | Coverage |
|-------|-------|-------|----------|
| **Unit** | Individual `RunPython` functions | `pytest`, `django.test` | 100% of data migrations |
| **Integration** | Full migration stack | `pytest-django`, test DB | All migrations forward/backward |
| **Contract** | Schema compatibility | `schemadiff`, `pg_dump` | Zero breaking changes |
| **Performance** | Large dataset | `pgbench`, custom scripts | <30 min for 10M rows |
| **Tenant Isolation** | RLS policies | Custom test suite | 100% tenant pairs |

### 9.2 Test Fixtures

```python
# backend/tests/fixtures/migration_test_case.py
class MigrationTestCase(TransactionTestCase):
    """Base test case for migration testing."""
    
    # Test both forward and reverse
    def test_migration_forward_and_reverse(self):
        """Test migration can be applied and rolled back."""
        from django.db import connection
        from django.db.migrations.executor import MigrationExecutor
        
        executor = MigrationExecutor(connection)
        
        # Migrate to before our migration
        executor.migrate([(self.app, self.migrate_from)])
        
        # Apply migration
        executor.migrate([(self.app, self.migrate_to)])
        
        # Verify schema state
        self.assertSchemaState(self.expected_state)
        
        # Rollback
        executor.migrate([(self.app, self.migrate_from)])
        
        # Verify rollback state
        self.assertSchemaState(self.rollback_state)
    
    def test_data_migration_idempotent(self):
        """Test data migration can be run multiple times safely."""
        executor = MigrationExecutor(connection)
        executor.migrate([(self.app, self.migrate_to)])
        
        # Capture state
        state1 = self.capture_data_state()
        
        # Re-run migration (simulate re-apply)
        executor.migrate([(self.app, self.migrate_from)])
        executor.migrate([(self.app, self.migrate_to)])
        
        state2 = self.capture_data_state()
        self.assertEqual(state1, state2)
    
    def test_tenant_isolation_rls(self):
        """Test RLS policies prevent cross-tenant access."""
        from django.db import connection
        
        # Create two tenant schemas
        tenant1 = self.create_test_tenant('tenant_1')
        tenant2 = self.create_test_tenant('tenant_2')
        
        # Insert data in both
        self.insert_test_data(tenant1, {'name': 'Asset 1'})
        self.insert_test_data(tenant2, {'name': 'Asset 2'})
        
        # Set search_path to tenant1
        with connection.cursor() as cursor:
            cursor.execute(f'SET search_path TO "{tenant1.schema_name}", public')
            cursor.execute('SELECT name FROM nxa_assets_asset')
            results = [row[0] for row in cursor.fetchall()]
        
        # Should only see tenant1 data
        self.assertEqual(results, ['Asset 1'])
```

### 9.3 Performance Benchmarks

```python
# backend/tests/performance/test_migration_performance.py
import pytest
from django.test import TestCase


class MigrationPerformanceTest(TestCase):
    """Performance benchmarks for critical migrations."""
    
    @pytest.mark.performance
    def test_backfill_10m_assets_under_30min(self):
        """Backfill 10M assets in <30 minutes."""
        from nxa_assets.migrations.0006_backfill_computed_fields import backfill_search_vector
        from django.db import connection
        from django.apps import apps
        
        # Create 10M test assets (using COPY for speed)
        self.create_test_assets(10_000_000)
        
        start = time.time()
        with connection.cursor() as cursor:
            # Simulate migration executor context
            backfill_search_vector(apps, connection.schema_editor())
        duration = time.time() - start
        
        # Should complete in <1800 seconds (30 min)
        self.assertLess(duration, 1800)
        self.assertEqual(Asset.objects.filter(search_vector__isnull=False).count(), 10_000_000)
    
    @pytest.mark.performance
    def test_tenant_provisioning_under_60s(self):
        """Tenant provisioning (schema + migrations + seed) < 60s."""
        from nxa_tenants.tasks.provisioning import provision_tenant
        
        tenant = Tenant.objects.create(
            name="Perf Test",
            tier="professional",
            admin_email="test@example.com"
        )
        
        start = time.time()
        provision_tenant(str(tenant.id))  # Sync for test
        duration = time.time() - start
        
        self.assertLess(duration, 60)
        self.assertEqual(tenant.status, 'active')
```

---

## 10. Monitoring & Alerting for Migrations

### 10.1 Migration Metrics

| Metric | Source | Alert Threshold | Dashboard |
|--------|--------|-----------------|-----------|
| `nxa_migration_duration_seconds` | Migration runner | > 1800s (30 min) | `svc-migrations-overview` |
| `nxa_migration_records_processed_total` | Migration runner | Stalled (no progress 10m) | `svc-migrations-overview` |
| `nxa_tenant_provisioning_duration_seconds` | Provisioning task | > 120s | `svc-tenant-onboarding` |
| `nxa_tenant_provisioning_failures_total` | Provisioning task | > 0 in 1h | `svc-tenant-onboarding` |
| `nxa_lansweeper_import_lag_seconds` | Sync job | > 3600s (1h) | `svc-lansweeper-sync` |
| `nxa_lansweeper_reconciliation_conflicts_total` | Reconciliation | > 100 in 1h | `svc-lansweeper-sync` |

### 10.2 Migration Dashboards

```json
{
  "dashboard": "svc-migrations-overview",
  "panels": [
    {
      "title": "Active Migrations",
      "type": "table",
      "targets": [
        {"expr": "nxa_migration_active{status=\"running\"}", "format": "table"}
      ]
    },
    {
      "title": "Migration Duration (p50/p95/p99)",
      "type": "timeseries",
      "targets": [
        {"expr": "histogram_quantile(0.50, rate(nxa_migration_duration_seconds_bucket[5m]))", "legendFormat": "p50"},
        {"expr": "histogram_quantile(0.95, rate(nxa_migration_duration_seconds_bucket[5m]))", "legendFormat": "p95"},
        {"expr": "histogram_quantile(0.99, rate(nxa_migration_duration_seconds_bucket[5m]))", "legendFormat": "p99"}
      ]
    },
    {
      "title": "Tenant Provisioning Success Rate",
      "type": "gauge",
      "targets": [
        {"expr": "sum(rate(nxa_tenant_provisioning_success_total[1h])) / sum(rate(nxa_tenant_provisioning_total[1h]))", "legendFormat": "Success Rate"}
      ]
    }
  ]
}
```

---

## 11. Disaster Recovery for Migrations

### 11.1 Backup Strategy

| Data Type | Backup Method | Frequency | Retention | RPO | RTO |
|-----------|---------------|-----------|-----------|-----|-----|
| **Schema (migrations)** | Git (source of truth) | Continuous | Forever | 0 | 0 |
| **Reference Data** | Migration files + DB dump | Per release | 13 months | 0 | <5 min |
| **Tenant Data** | PgBaseBackup + WAL-G | Continuous (streaming) | 30d hot / 13mo cold | <1s | <15 min |
| **Lansweeper Raw** | S3 (immutable) | Per sync job | 90 days | 0 | <1 hour |
| **Migration State** | `django_migrations` table | Continuous | Forever | 0 | <5 min |

### 11.2 Point-in-Time Recovery (PITR)

```bash
# scripts/pitr_recovery.sh
# Restore tenant schema to specific timestamp

TENANT_ID=$1
TARGET_TIME=$2  # ISO8601, e.g., "2026-11-09T12:00:00Z"
SCHEMA_NAME="tenant_$(echo $TENANT_ID | tr '-' '_')"

# 1. Create recovery schema
psql -c "CREATE SCHEMA \"${SCHEMA_NAME}_recovery\""

# 2. Restore from WAL-G at target time
wal-g backup-fetch \
  --target-time="${TARGET_TIME}" \
  --schema="${SCHEMA_NAME}" \
  --target-schema="${SCHEMA_NAME}_recovery"

# 3. Validate data integrity
./scripts/validate_tenant_schema.sh "${SCHEMA_NAME}_recovery"

# 4. Swap schemas (atomic)
psql -c "
  ALTER SCHEMA \"${SCHEMA_NAME}\" RENAME TO \"${SCHEMA_NAME}_corrupted\";
  ALTER SCHEMA \"${SCHEMA_NAME}_recovery\" RENAME TO \"${SCHEMA_NAME}\";
"

# 5. Update tenant record
psql -c "
  UPDATE nxa_tenants_tenant 
  SET schema_name = '${SCHEMA_NAME}', 
      restored_at = NOW(),
      restored_from = '${TARGET_TIME}'
  WHERE id = '${TENANT_ID}';
"
```

---

## 12. Compliance & Audit

### 12.1 Migration Audit Trail

```python
# backend/nexusasset/nxa_core/models/migration_audit.py

class MigrationAuditLog(models.Model):
    """Immutable audit log for all migration operations."""
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid7)
    timestamp = models.DateTimeField(auto_now_add=True, db_index=True)
    
    # Migration identity
    app_label = models.CharField(max_length=100, db_index=True)
    migration_name = models.CharField(max_length=255)
    migration_direction = models.CharField(
        max_length=10, 
        choices=[('forward', 'Forward'), ('reverse', 'Reverse')]
    )
    
    # Execution context
    executed_by = models.CharField(max_length=255)  # user, CI job, system
    execution_id = models.CharField(max_length=100)  # CI run ID, task ID
    environment = models.CharField(max_length=20, choices=ENV_CHOICES)
    
    # Tenant scope (null = global)
    tenant_id = models.UUIDField(null=True, db_index=True)
    schema_name = models.CharField(max_length=100, null=True)
    
    # Results
    status = models.CharField(max_length=20, choices=STATUS_CHOICES)
    duration_ms = models.PositiveIntegerField()
    records_affected = models.PositiveBigIntegerField(default=0)
    error_message = models.TextField(null=True)
    
    # Checksums for verification
    pre_schema_checksum = models.CharField(max_length=64, null=True)
    post_schema_checksum = models.CharField(max_length=64, null=True)
    data_checksum = models.CharField(max_length=64, null=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['app_label', 'migration_name', 'timestamp']),
            models.Index(fields=['tenant_id', 'timestamp']),
            models.Index(fields=['environment', 'status', 'timestamp']),
        ]
        ordering = ['-timestamp']
```

### 12.2 Compliance Reports

| Report | Frequency | Audience | Content |
|--------|-----------|----------|---------|
| **Migration Compliance** | Per release | Security, Audit | All migrations applied, checksums verified, rollback tested |
| **Tenant Provisioning Audit** | Monthly | Compliance | All tenant creations/deletions, GDPR erasures, access logs |
| **Data Integrity Verification** | Weekly | Data Engineering | Checksums, row counts, referential integrity, RLS validation |
| **Lansweeper Sync Audit** | Daily | ITAM Team | Import completeness, conflict resolution log, drift detection |

---

## 13. Future Enhancements (Roadmap)

| Quarter | Initiative | Description |
|---------|------------|-------------|
| **Q1 2027** | **Declarative Migrations** | Atlas/Ent schema-as-code, auto-generate Django migrations |
| **Q2 2027** | **Online Schema Changes** | `pg_repack` / `pg_squeeze` for zero-downtime ALTER TABLE |
| **Q2 2027** | **Migration Canary** | Run migration on 1% tenant shard before full rollout |
| **Q3 2027** | **Automated Rollback ML** | Anomaly detection on migration metrics → auto-rollback |
| **Q3 2027** | **Cross-Cloud Sync** | Multi-region active-active with conflict-free replicated data types |
| **Q4 2027** | **Time-Travel Queries** | `AS OF TIMESTAMP` support for tenant point-in-time recovery |

---

## 14. References

| Doc ID | Title | Link |
|--------|-------|------|
| DOC-04 | Domain Model | [04-domain-model.md](../00-overview/04-domain-model.md) |
| DOC-05 | Application Architecture | [05-application-architecture.md](../00-overview/05-application-architecture.md) |
| DOC-06 | Infrastructure Architecture | [06-infrastructure-architecture.md](../00-overview/06-infrastructure-architecture.md) |
| DOC-07 | API Design | [07-api-design.md](../00-overview/07-api-design.md) |
| DOC-08 | Lansweeper Integration | [08-lansweeper-integration.md](../00-overview/08-lansweeper-integration.md) |
| DOC-11 | DevOps & CI/CD | [11-devops-cicd.md](../00-overview/11-devops-cicd.md) |
| ADR-025 | Migration Strategy | [ADR-025](../14-appendices/adr-index.md#adr-025) |
| ADR-026 | Tenant Isolation (RLS) | [ADR-026](../14-appendices/adr-index.md#adr-026) |
| ADR-027 | Seed Data Management | [ADR-027](../14-appendices/adr-index.md#adr-027) |
| ADR-028 | Lansweeper Reconciliation | [ADR-028](../14-appendices/adr-index.md#adr-028) |
| ADR-029 | Rollback Procedures | [ADR-029](../14-appendices/adr-index.md#adr-029) |
| ADR-030 | Disaster Recovery | [ADR-030](../14-appendices/adr-index.md#adr-030) |

---

> **Document Control:** This document is maintained by Data Engineering. Changes require review by Platform Lead and Architecture Review Board. Next review: 2027-02-09.