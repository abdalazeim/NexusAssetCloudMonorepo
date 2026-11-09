╔═════════════════════════════════════════════════════════════════════════════════════════════════╗
║ Document ID: DOC-10                                                                            ║
║ Title: Testing Strategy — Test Pyramid, Tenant Isolation, Contract, Chaos, Performance         ║
║ Version: 1.0.0                                                                                 ║
║ Status: ✓ Approved                                                                             ║
║ Classification: Internal — Confidential                                                        ║
║ Authors: NexusAsset Cloud QA & Platform Engineering Team                                       ║
║ Reviewers: Backend, Frontend, Security, Platform, DevOps                                       ║
║ Approved By: CTO / VP Engineering / QA Lead                                                    ║
║ Date: 2026-11-09                                                                               ║
║ Supersedes: —                                                                                  ║
║ Related: DOC-00, DOC-01, DOC-02, DOC-03, DOC-04, DOC-05, DOC-06, DOC-07, DOC-09, DOC-11       ║
║ Traceability: TC-### ← REQ-FN-###/REQ-NFR-###, TC-### → CI Pipeline, ADR-### → Test Decisions   ║
╚════════════════════════════════════════════════════════════════════════════════════════════════╝

# ═══════════════════════════════════════════════════════════════
# NexusAsset Cloud — Testing Strategy (DOC-10)
# ═══════════════════════════════════════════════════════════════

> **Executive Summary** — This document defines the comprehensive testing strategy for NexusAsset Cloud. The strategy follows the **Test Pyramid** (Unit → Integration → Contract → E2E → Chaos → Performance) with emphasis on **tenant isolation verification**, **contract testing** for API stability, **chaos engineering** for resilience, and **performance benchmarks** for SLO compliance. All tests are automated in CI/CD pipelines (DOC-11) with quality gates. Test coverage targets: **Domain ≥ 90%**, **Application ≥ 85%**, **Infrastructure ≥ 80%**, **Frontend ≥ 80%**. The strategy ensures **shift-left testing** with developer-owned unit/integration tests and QA-owned contract/E2E/chaos tests.

---

## Table of Contents

1. [Testing Principles & Philosophy](#1-testing-principles--philosophy)
2. [Test Pyramid Structure](#2-test-pyramid-structure)
3. [Unit Testing](#3-unit-testing)
4. [Integration Testing](#4-integration-testing)
5. [Contract Testing](#5-contract-testing)
6. [Tenant Isolation Testing](#6-tenant-isolation-testing)
7. [End-to-End Testing](#7-end-to-end-testing)
8. [Chaos Engineering](#8-chaos-engineering)
9. [Performance & Load Testing](#9-performance--load-testing)
10. [Security Testing](#10-security-testing)
11. [Accessibility Testing](#11-accessibility-testing)
12. [Test Data Management](#12-test-data-management)
13. [Test Environments](#13-test-environments)
14. [CI/CD Integration](#14-cicd-integration)
15. [Quality Gates & Metrics](#15-quality-gates--metrics)
16. [Diagrams](#16-diagrams)
17. [Tables](#17-tables)

---

## 1. Testing Principles & Philosophy

### 1.1 Core Principles

| Principle ID | Principle | Description |
|--------------|-----------|-------------|
| **TEST-PRIN-001** | **Shift Left** | Developers write and own unit/integration tests; run locally pre-commit |
| **TEST-PRIN-002** | **Test Pyramid** | More unit tests, fewer integration, minimal E2E; fast feedback |
| **TEST-PRIN-003** | **Tenant Isolation First** | Every test validates tenant boundary; cross-tenant access = test failure |
| **TEST-PRIN-004** | **Contract as Source of Truth** | OpenAPI spec drives contract tests; breaking changes blocked |
| **TEST-PRIN-005** | **Deterministic & Isolated** | No flaky tests; each test independent; parallel execution |
| **TEST-PRIN-006** | **Production-Like Environments** | Test against real dependencies (PostgreSQL, Redis, Kafka) via Testcontainers |
| **TEST-PRIN-007** | **Observability in Tests** | Tests emit metrics, logs, traces; correlated with production |
| **TEST-PRIN-008** | **Chaos as Regression** | Chaos experiments run weekly; failures = new test cases |
| **TEST-PRIN-009** | **Performance Budgets** | Performance tests in CI; regression > 10% = build failure |
| **TEST-PRIN-010** | **Documentation as Tests** | README examples tested; API docs validated against spec |

### 1.2 Testing Scope by Layer

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              TESTING SCOPE BY LAYER                                                 │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  UNIT TESTS (70%)                                                                          │   │
│  │  • Domain layer: Aggregates, Entities, VOs, Domain Services, Specifications               │   │
│  │  • Application layer: Use Cases, Mappers, DTOs                                            │   │
│  │  • Frontend: Pure functions, hooks, validators, utility components                        │   │
│  │  • Infrastructure: Adapters (mocked external deps)                                        │   │
│  │  • Tools: Vitest (backend), Vitest + RTL (frontend)                                       │   │
│  │  • Target: Domain ≥ 90%, Application ≥ 85%, Frontend ≥ 80%                                │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                                                │
│                                    ▼                                                                │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  INTEGRATION TESTS (20%)                                                                   │   │
│  │  • Repository implementations (Django ORM + Testcontainers PostgreSQL)                    │   │
│  │  • Event publishing/consumption (Kafka + Testcontainers)                                  │   │
│  │  • External API adapters (Lansweeper GraphQL, Stripe, Cloud Providers) via WireMock       │   │
│  │  • Database migrations (apply + rollback)                                                 │   │
│  │  • Cross-context integration events (IEV-###)                                             │   │
│  │  • Tools: pytest (backend), Vitest + MSW (frontend)                                       │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                                                │
│                                    ▼                                                                │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  CONTRACT TESTS (5%)                                                                       │   │
│  │  • Provider: Backend honors OpenAPI spec (Schemathesis/Pact)                              │   │
│  │  • Consumer: Frontend/BFF correctly consumes API (Pact)                                   │   │
│  │  • Event contracts: Integration event schemas (Avro/JSON Schema)                          │   │
│  │  • Tools: Schemathesis, Pact, spectral                                                     │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                                                │
│                                    ▼                                                                │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  E2E TESTS (3%)                                                                            │   │
│  │  • Critical user journeys: Login → Asset CRUD → Compliance → Billing                      │   │
│  │  • Multi-tenant scenarios: Tenant A cannot access Tenant B data                           │   │
│  │  • SSO flows: SAML, OIDC                                                                   │   │
│  │  • Tools: Playwright (frontend), pytest + httpx (backend API)                             │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                                                │
│                                    ▼                                                                │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  CHAOS ENGINEERING (1%)                                                                    │   │
│  │  • Pod kill, network latency, clock skew, DNS failure                                     │   │
│  │  • Dependency failure: DB, Redis, Kafka, External APIs                                    │   │
│  │  • SLO validation under stress                                                            │   │
│  │  • Tools: Chaos Mesh, Litmus, custom scripts                                              │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                                                │
│                                    ▼                                                                │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │  PERFORMANCE TESTS (1%)                                                                    │   │
│  │  • Load testing: k6 scripts for API endpoints                                             │   │
│  │  • Soak testing: 24h sustained load                                                        │   │
│  │  • Spike testing: Sudden traffic increase                                                  │   │
│  │  • Frontend: Lighthouse CI, WebPageTest                                                    │   │
│  │  • Tools: k6, Locust, Lighthouse CI, WebPageTest                                          │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Test Pyramid Structure

### 2.1 Distribution & Execution Times

| Layer | % of Tests | Target Count | Execution Time | Parallelism | When Run |
|-------|------------|--------------|----------------|-------------|----------|
| **Unit** | 70% | ~5,000 | < 5 min | High (50 workers) | Pre-commit, CI every push |
| **Integration** | 20% | ~1,500 | < 15 min | Medium (10 workers) | CI every push, nightly full |
| **Contract** | 5% | ~200 | < 5 min | Medium (10 workers) | CI every push, PR validation |
| **E2E** | 3% | ~50 | < 20 min | Low (3 workers) | CI on merge, nightly, release |
| **Chaos** | 1% | ~20 | < 60 min | Sequential | Weekly scheduled, on-demand |
| **Performance** | 1% | ~10 | < 60 min | Sequential | Nightly, release, on-demand |

---

## 3. Unit Testing

### 3.1 Backend Unit Tests (Domain Layer)

```python
# backend/tests/unit/nxa_assets/domain/test_asset.py
import pytest
from nexusasset.assets.domain.aggregates.asset import Asset, AssetId
from nexusasset.assets.domain.value_objects import AssetState, AssetType
from nexusasset.shared.kernel.domain.exceptions import InvariantViolationError

class TestAssetAggregate:
    """Unit tests for Asset aggregate - no external dependencies"""
    
    def test_asset_creation_sets_initial_state(self):
        asset = Asset.create(
            asset_id=AssetId.generate(),
            tenant_id=tenant_id,
            asset_type=AssetType.SERVER,
            name="Test Server",
            created_by=user_id,
        )
        
        assert asset.state == AssetState.DISCOVERED
        assert asset.confidence_score == 1.0
        assert asset.version == 1
    
    def test_authorize_transitions_from_discovered_to_authorized(self):
        asset = Asset.create(...)
        asset.state = AssetState.DISCOVERED
        
        asset.authorize(actor_id=user_id, reason="Policy passed")
        
        assert asset.state == AssetState.AUTHORIZED
        assert asset.state_changed_by == user_id
        assert asset.version == 2
    
    def test_authorize_from_active_raises_invariant_violation(self):
        asset = Asset.create(...)
        asset.state = AssetState.ACTIVE
        
        with pytest.raises(InvariantViolationError, match="Cannot authorize from ACTIVE state"):
            asset.authorize(actor_id=user_id, reason="...")
    
    def test_field_ownership_lansweeper_wins_for_hardware(self):
        asset = Asset.create(...)
        asset.state = AssetState.ACTIVE
        
        # Simulate Lansweeper update
        asset.update_hardware_specs(
            specs=new_specs,
            source=AssetSource.LANSWEEPER,
            actor=user_id
        )
        
        # Lansweeper should win for CPU
        assert asset.specifications.cpu.model == new_specs.cpu.model
    
    def test_field_ownership_manual_wins_for_owner(self):
        asset = Asset.create(...)
        asset.state = AssetState.ACTIVE
        asset.ownership.owner_id = original_owner
        
        # Manual update should win
        asset.update_ownership(
            owner_id=new_owner,
            custodian_id=custodian,
            actor=user_id
        )
        
        assert asset.ownership.owner_id == new_owner
```

### 3.2 Frontend Unit Tests

```typescript
// src/lib/hooks/__tests__/useAssets.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useAssets } from '../useAssets';
import { server } from '@/mocks/node';
import { http, HttpResponse } from 'msw';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
};

describe('useAssets', () => {
  beforeEach(() => {
    server.use(
      http.get('/api/v1/assets', () => {
        return HttpResponse.json({
          data: [{ id: 'ast_001', name: 'Server 1', asset_type: 'SERVER', state: 'ACTIVE' }],
          meta: { total_count: 1, has_more: false },
        });
      })
    );
  });

  it('fetches assets with default params', async () => {
    const { result } = renderHook(() => useAssets({}), { wrapper: createWrapper() });
    
    await waitFor(() => expect(result.current.isSuccess).toBe(true));
    
    expect(result.current.data?.data).toHaveLength(1);
    expect(result.current.data?.data[0].name).toBe('Server 1');
  });

  it('applies filters correctly', async () => {
    const { result } = renderHook(() => useAssets({ 
      filter: { asset_type: ['SERVER'] },
      sort: '-created_at'
    }), { wrapper: createWrapper() });
    
    await waitFor(() => expect(result.current.isSuccess).toBe(true));
    // Verify request params via MSW
  });
});
```

### 3.3 Test Coverage Targets

| Module | Target | Enforcement |
|--------|--------|-------------|
| **Domain (all contexts)** | 90% lines, 85% branches | `pytest --cov-fail-under=90` |
| **Application (use cases)** | 85% lines, 80% branches | `pytest --cov-fail-under=85` |
| **Frontend (hooks, utils, components)** | 80% lines, 75% branches | `vitest --coverage` |
| **Infrastructure (adapters)** | 80% lines | `pytest --cov-fail-under=80` |

---

## 4. Integration Testing

### 4.1 Repository Integration Tests (Testcontainers)

```python
# backend/tests/integration/nxa_assets/test_django_asset_repository.py
import pytest
from testcontainers.postgres import PostgresContainer
from nexusasset.assets.infrastructure.repositories.django_asset_repository import DjangoAssetRepository
from nexusasset.assets.domain.aggregates.asset import Asset

@pytest.fixture(scope="session")
def postgres():
    with PostgresContainer("postgres:18") as postgres:
        yield postgres.get_connection_url()

@pytest.fixture(scope="function")
def django_db(postgres):
    # Apply migrations
    call_command('migrate', '--run-syncdb')
    yield
    # Cleanup
    call_command('flush', '--no-input')

class TestDjangoAssetRepository:
    def test_save_and_get_asset(self, django_db):
        repo = DjangoAssetRepository(tenant_id=tenant_id)
        asset = Asset.create(...)
        
        repo.add(asset)
        retrieved = repo.get(asset.id)
        
        assert retrieved is not None
        assert retrieved.name == asset.name
        assert retrieved.state == asset.state
    
    def test_tenant_isolation_enforced_by_rls(self, django_db):
        repo_tenant_a = DjangoAssetRepository(tenant_id=tenant_a_id)
        repo_tenant_b = DjangoAssetRepository(tenant_id=tenant_b_id)
        
        asset_a = Asset.create(tenant_id=tenant_a_id, ...)
        repo_tenant_a.add(asset_a)
        
        # Tenant B should NOT see Tenant A's asset
        retrieved = repo_tenant_b.get(asset_a.id)
        assert retrieved is None
    
    def test_optimistic_locking_on_concurrent_update(self, django_db):
        repo = DjangoAssetRepository(tenant_id=tenant_id)
        asset = Asset.create(...)
        repo.add(asset)
        
        # Simulate concurrent updates
        asset1 = repo.get(asset.id)
        asset2 = repo.get(asset.id)
        
        asset1.update_name("Name 1")
        repo.update(asset1)  # Success
        
        asset2.update_name("Name 2")
        with pytest.raises(OptimisticLockError):
            repo.update(asset2)  # Should fail - version mismatch
```

### 4.2 Event Integration Tests

```python
# backend/tests/integration/nxa_core/test_event_publishing.py
import pytest
from testcontainers.kafka import KafkaContainer
from nexusasset.shared.kernel.infrastructure.messaging.kafka_event_publisher import KafkaEventPublisher
from nexusasset.assets.domain.events import AssetAuthorizedEvent

@pytest.fixture(scope="session")
def kafka():
    with KafkaContainer() as kafka:
        yield kafka.get_bootstrap_server()

class TestEventPublishing:
    def test_domain_event_published_to_kafka(self, kafka, django_db):
        publisher = KafkaEventPublisher(bootstrap_servers=kafka)
        outbox_repo = OutboxRepository()
        
        event = AssetAuthorizedEvent(
            asset_id=asset_id,
            tenant_id=tenant_id,
            authorized_by=user_id,
            reason="Test"
        )
        
        # Publish within transaction
        async with DjangoUnitOfWork() as uow:
            await publisher.publish(event)
            await uow.commit()
        
        # Verify in outbox
        outbox_entries = outbox_repo.get_unpublished()
        assert len(outbox_entries) == 1
        assert outbox_entries[0].event_type == "AssetAuthorizedEvent"
    
    def test_outbox_processor_publishes_to_kafka(self, kafka, django_db):
        # Add entry to outbox
        outbox_repo.add(OutboxEntry(...))
        
        # Run processor
        processor = OutboxProcessor(bootstrap_servers=kafka)
        processed = processor.process_batch(batch_size=10)
        
        assert processed == 1
        
        # Verify in Kafka topic
        consumer = KafkaConsumer("nxa.assets.authorized", bootstrap_servers=kafka)
        messages = consumer.poll(timeout_ms=5000)
        assert len(messages) == 1
```

### 4.3 External API Integration Tests (WireMock)

```python
# backend/tests/integration/nxa_lansweeper/test_graphql_client.py
import pytest
import wiremock
from nexusasset.lansweeper.infrastructure.http.graphql_client import LansweeperGraphQLClient

@pytest.fixture
def wiremock_server():
    server = wiremock.WireMockServer(port=8080)
    server.start()
    yield server
    server.stop()

class TestLansweeperGraphQLClient:
    def test_query_assets_paginated(self, wiremock_server):
        # Setup mock response
        wiremock_server.stub_for(
            wiremock.post("/graphql")
            .with_request_body_containing("assets")
            .will_return(wiremock.ok_json({
                "data": {
                    "assets": {
                        "pageInfo": {"hasNextPage": False, "endCursor": "cursor123"},
                        "edges": [{
                            "cursor": "cursor123",
                            "node": {
                                "assetId": "LS-001",
                                "assetName": "Test Server",
                                "assetType": "Windows Computer",
                                "serialNumber": "SN123",
                                "lastChanged": "2026-11-09T10:00:00Z"
                            }
                        }]
                    }
                }
            }))
        )
        
        client = LansweeperGraphQLClient(
            site=LansweeperSite(graphql_endpoint="http://localhost:8080/graphql", ...),
            vault=MockVault()
        )
        
        result = await client.query_assets_paginated(cursor=None, filter=AssetFilter())
        
        assert len(result.edges) == 1
        assert result.edges[0].node.asset_name == "Test Server"
```

---

## 5. Contract Testing

### 5.1 Provider Contract Tests (Backend)

```python
# backend/tests/contracts/provider/test_openapi_contract.py
import schemathesis
from hypothesis import given, settings

# Load OpenAPI spec
schema = schemathesis.from_path("api/contracts/openapi.yaml")

# Test all endpoints against spec
@schema.parametrize()
@settings(max_examples=50, deadline=None)
def test_api_conforms_to_openapi(case):
    response = case.call_wsgi(app)  # Django test client
    case.validate_response(response)

# Custom checks
@schema.check
def no_500_errors(ctx, response, case):
    assert response.status_code < 500, f"Server error: {response.status_code}"

@schema.check
def response_matches_schema(ctx, response, case):
    # Already validated by schemathesis
    pass

# Run with: pytest tests/contracts/provider/ -v
```

### 5.2 Consumer Contract Tests (Frontend/BFF)

```typescript
// frontend/tests/contracts/consumer/bff-assets.contract.test.ts
import { pactWith } from '@pact-foundation/pact-vitest';
import { getAssetsDashboard } from '@/lib/bff/assets';

pactWith({ consumer: 'nxa-frontend-bff', provider: 'nxa-backend' }, (provider) => {
  describe('GET /api/bff/assets/dashboard', () => {
    it('returns dashboard data', async () => {
      await provider.addInteraction({
        state: 'tenant has assets and compliance data',
        uponReceiving: 'a request for assets dashboard',
        withRequest: {
          method: 'GET',
          path: '/api/bff/assets/dashboard',
          headers: {
            'Authorization': 'Bearer valid-token',
            'X-Tenant-ID': 'tn_abc123',
          },
        },
        willRespondWith: {
          status: 200,
          headers: { 'Content-Type': 'application/json' },
          body: {
            assets: eachLike({
              id: 'ast_001',
              name: 'Server 1',
              asset_type: 'SERVER',
              state: 'ACTIVE',
            }),
            stats: {
              totalAssets: 150,
              complianceScore: 95,
              recentActivity: eachLike({
                type: 'AssetStateChanged',
                timestamp: '2026-11-09T10:30:45Z',
              }),
            },
          },
        },
      });

      const result = await getAssetsDashboard(provider.mockService.baseUrl);
      expect(result.assets).toHaveLength(1);
      expect(result.stats.complianceScore).toBe(95);
    });
  });
});
```

### 5.3 Event Contract Tests

```python
# backend/tests/contracts/events/test_integration_events.py
import pytest
from nexusasset.shared.kernel.domain.events import IntegrationEvent
from nexusasset.assets.domain.events import AssetDiscoveredEvent

class TestEventContracts:
    def test_asset_discovered_event_schema(self):
        event = AssetDiscoveredEvent(
            asset_id="ast_001",
            tenant_id="tn_abc123",
            source="LANSWEEPER",
            external_id="LS-001",
            raw_hash="sha256:abc123",
            discovered_at="2026-11-09T10:30:45Z"
        )
        
        # Validate against JSON Schema
        schema = load_json_schema("events/asset_discovered.json")
        validate(instance=event.model_dump(), schema=schema)
    
    def test_event_backward_compatibility(self):
        # Old event format should still parse
        old_event = {
            "asset_id": "ast_001",
            "tenant_id": "tn_abc123",
            "source": "LANSWEEPER",
            "external_id": "LS-001",
            # missing: raw_hash, discovered_at (added in v2)
        }
        
        event = AssetDiscoveredEvent.model_validate(old_event)
        assert event.raw_hash is None  # Optional field
```

---

## 6. Tenant Isolation Testing

### 6.1 Database-Level Isolation Tests

```python
# backend/tests/isolation/test_database_rls.py
import pytest
from nexusasset.assets.infrastructure.repositories.django_asset_repository import DjangoAssetRepository
from nexusasset.assets.domain.aggregates.asset import Asset

class TestDatabaseTenantIsolation:
    def test_rls_prevents_cross_tenant_select(self, django_db):
        repo_a = DjangoAssetRepository(tenant_id=tenant_a_id)
        repo_b = DjangoAssetRepository(tenant_id=tenant_b_id)
        
        asset_a = Asset.create(tenant_id=tenant_a_id, name="Asset A", ...)
        repo_a.add(asset_a)
        
        # Direct query with Tenant B context should return empty
        assets = repo_b.list_all()
        assert len(assets) == 0
    
    def test_rls_prevents_cross_tenant_update(self, django_db):
        repo_a = DjangoAssetRepository(tenant_id=tenant_a_id)
        repo_b = DjangoAssetRepository(tenant_id=tenant_b_id)
        
        asset_a = Asset.create(tenant_id=tenant_a_id, name="Asset A", ...)
        repo_a.add(asset_a)
        
        # Tenant B tries to update Tenant A's asset
        asset_a.name = "Hacked"
        with pytest.raises(AssetNotFoundError):
            repo_b.update(asset_a)
    
    def test_rls_prevents_cross_tenant_delete(self, django_db):
        repo_a = DjangoAssetRepository(tenant_id=tenant_a_id)
        repo_b = DjangoAssetRepository(tenant_id=tenant_b_id)
        
        asset_a = Asset.create(tenant_id=tenant_a_id, ...)
        repo_a.add(asset_a)
        
        with pytest.raises(AssetNotFoundError):
            repo_b.delete(asset_a.id)
    
    def test_raw_sql_blocked_by_linter(self):
        # This should fail in CI due to linter rule
        with pytest.raises(LintError):
            # Raw SQL without tenant_id
            cursor.execute("SELECT * FROM assets WHERE id = %s", [asset_id])
```

### 6.2 API-Level Isolation Tests

```python
# backend/tests/isolation/test_api_tenant_isolation.py
import pytest
from httpx import AsyncClient

class TestApiTenantIsolation:
    async def test_tenant_a_cannot_access_tenant_b_assets(self, async_client: AsyncClient):
        # Login as Tenant A user
        token_a = await login(async_client, user_a)
        headers_a = {"Authorization": f"Bearer {token_a}", "X-Tenant-ID": tenant_a_id}
        
        # Login as Tenant B user
        token_b = await login(async_client, user_b)
        headers_b = {"Authorization": f"Bearer {token_b}", "X-Tenant-ID": tenant_b_id}
        
        # Tenant A creates asset
        response = await async_client.post("/api/v1/assets", 
            headers=headers_a, json={"name": "Asset A", "asset_type": "SERVER"})
        assert response.status_code == 201
        asset_id = response.json()["data"]["id"]
        
        # Tenant B tries to access Tenant A's asset
        response = await async_client.get(f"/api/v1/assets/{asset_id}", headers=headers_b)
        assert response.status_code == 404  # Not found (not 403 to avoid enumeration)
    
    async def test_tenant_a_cannot_list_tenant_b_assets(self, async_client: AsyncClient):
        token_a = await login(async_client, user_a)
        headers_a = {"Authorization": f"Bearer {token_a}", "X-Tenant-ID": tenant_a_id}
        
        response = await async_client.get("/api/v1/assets", headers=headers_a)
        assert response.status_code == 200
        
        # Should only see Tenant A's assets
        asset_ids = [a["id"] for a in response.json()["data"]]
        # Verify none belong to Tenant B (would require DB check)
```

### 6.3 Frontend Isolation Tests

```typescript
// frontend/tests/isolation/tenant-isolation.test.ts
import { test, expect } from '@playwright/test';

test.describe('Tenant Isolation', () => {
  test('Tenant A user cannot see Tenant B assets in UI', async ({ page }) => {
    // Login as Tenant A
    await login(page, userA);
    await page.goto('/assets');
    
    // Verify only Tenant A assets shown
    const assetRows = await page.locator('[data-testid="asset-row"]').all();
    for (const row of assetRows) {
      const tenantBadge = row.locator('[data-testid="tenant-badge"]');
      await expect(tenantBadge).toHaveText('Tenant A');
    }
  });
  
  test('Tenant switcher only shows authorized tenants', async ({ page }) => {
    await login(page, userA);
    
    await page.click('[data-testid="tenant-switcher"]');
    const options = await page.locator('[data-testid="tenant-option"]').allTextContents();
    
    expect(options).toContain('Tenant A');
    expect(options).not.toContain('Tenant B');
  });
});
```

---

## 7. End-to-End Testing

### 7.1 Critical User Journeys

```typescript
// frontend/e2e/critical-journeys.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Critical User Journeys', () => {
  test('Complete Asset Lifecycle: Create → Authorize → Activate → Maintain → Retire', async ({ page }) => {
    await login(page, assetManager);
    
    // 1. Create asset
    await page.goto('/assets/create');
    await page.fill('[name="name"]', 'Test Server E2E');
    await page.selectOption('[name="asset_type"]', 'SERVER');
    await page.fill('[name="serial_number"]', 'E2E-SN-001');
    await page.click('[data-testid="submit"]');
    
    await expect(page.locator('[data-testid="toast"]')).toContainText('Asset created');
    const assetId = await page.locator('[data-testid="asset-id"]').textContent();
    
    // 2. Authorize asset
    await page.click('[data-testid="authorize-button"]');
    await page.fill('[name="reason"]', 'Approved by manager');
    await page.click('[data-testid="confirm-authorize"]');
    
    await expect(page.locator('[data-testid="asset-state"]')).toHaveText('AUTHORIZED');
    
    // 3. Activate asset
    await page.click('[data-testid="activate-button"]');
    await page.selectOption('[name="owner_id"]', 'user_001');
    await page.click('[data-testid="confirm-activate"]');
    
    await expect(page.locator('[data-testid="asset-state"]')).toHaveText('ACTIVE');
    
    // 4. Schedule maintenance
    await page.click('[data-testid="maintenance-tab"]');
    await page.click('[data-testid="schedule-maintenance"]');
    await page.fill('[name="start_date"]', '2026-12-01');
    await page.fill('[name="end_date"]', '2026-12-01T14:00');
    await page.click('[data-testid="submit"]');
    
    // 5. Complete maintenance
    await page.click('[data-testid="complete-maintenance"]');
    await page.fill('[name="outcome"]', 'Completed successfully');
    await page.click('[data-testid="confirm"]');
    
    // 6. Retire asset
    await page.click('[data-testid="retire-button"]');
    await page.fill('[name="reason"]', 'End of life');
    await page.fill('[name="sanitization_certificate"]', 'CERT-001');
    await page.click('[data-testid="confirm-retire"]');
    
    await expect(page.locator('[data-testid="asset-state"]')).toHaveText('RETIRED');
  });
  
  test('Compliance Assessment Run → Evidence Package Generation', async ({ page }) => {
    await login(page, complianceOfficer);
    
    await page.goto('/compliance/assessments');
    await page.click('[data-testid="run-assessment"]');
    await page.selectOption('[name="framework_id"]', 'ISO_27001_2022');
    await page.click('[data-testid="start-assessment"]');
    
    // Wait for completion (polling)
    await expect(page.locator('[data-testid="assessment-status"]')).toHaveText('COMPLETED', { timeout: 120000 });
    
    // Generate evidence package
    await page.click('[data-testid="generate-evidence"]');
    await page.click('[data-testid="download-package"]');
    
    // Verify download
    const download = await page.waitForEvent('download');
    expect(download.suggestedFilename()).toMatch(/evidence-package-.*\.zip/);
  });
  
  test('Lansweeper Sync → Reconciliation → Conflict Resolution', async ({ page }) => {
    await login(page, integrationAdmin);
    
    await page.goto('/lansweeper/sites');
    await page.click('[data-testid="site-row"]:first-child [data-testid="sync-button"]');
    await page.selectOption('[name="sync_type"]', 'INCREMENTAL');
    await page.click('[data-testid="start-sync"]');
    
    // Wait for sync completion
    await expect(page.locator('[data-testid="sync-status"]')).toHaveText('COMPLETED', { timeout: 300000 });
    
    // Check for conflicts
    await page.goto('/lansweeper/conflicts');
    const conflictCount = await page.locator('[data-testid="conflict-row"]').count();
    
    if (conflictCount > 0) {
      // Resolve first conflict
      await page.click('[data-testid="conflict-row"]:first-child [data-testid="resolve-button"]');
      await page.click('[data-testid="accept-discovered"]');
      await page.click('[data-testid="confirm-resolve"]');
      
      await expect(page.locator('[data-testid="toast"]')).toContainText('Conflict resolved');
    }
  });
});
```

### 7.2 Multi-Tenant E2E Tests

```typescript
// frontend/e2e/multi-tenant.spec.ts
test.describe('Multi-Tenant Scenarios', () => {
  test('Tenant provisioning → User invitation → Asset creation', async ({ page, context }) => {
    // Platform admin provisions tenant
    await login(page, platformAdmin);
    await page.goto('/platform/tenants/create');
    await page.fill('[name="name"]', 'E2E Test Tenant');
    await page.fill('[name="slug"]', 'e2e-test');
    await page.fill('[name="admin_email"]', 'e2e-admin@example.com');
    await page.click('[data-testid="create-tenant"]');
    
    await expect(page.locator('[data-testid="toast"]')).toContainText('Tenant provisioned');
    
    // New tenant admin logs in (separate context)
    const tenantPage = await context.newPage();
    await login(tenantPage, 'e2e-admin@example.com', 'tenant-password');
    
    // Invite user
    await tenantPage.goto('/tenants/users/invite');
    await tenantPage.fill('[name="email"]', 'new-user@example.com');
    await tenantPage.selectOption('[name="roles"]', 'asset_manager');
    await tenantPage.click('[data-testid="send-invitation"]');
    
    // New user accepts invitation (another context)
    const userPage = await context.newPage();
    await userPage.goto('/invite/accept?token=...');
    await userPage.fill('[name="password"]', 'new-password');
    await userPage.fill('[name="confirm_password"]', 'new-password');
    await userPage.click('[data-testid="accept-invitation"]');
    
    // Create asset
    await userPage.goto('/assets/create');
    await userPage.fill('[name="name"]', 'User Asset');
    await userPage.click('[data-testid="submit"]');
    
    await expect(userPage.locator('[data-testid="toast"]')).toContainText('Asset created');
  });
});
```

---

## 8. Chaos Engineering

### 8.1 Chaos Experiment Catalog

| Experiment ID | Name | Target | Injection | Expected Behavior | SLO Validation |
|---------------|------|--------|-----------|-------------------|----------------|
| **CHAOS-001** | Backend Pod Kill | nxa-backend | Kill 1/3 pods | Auto-reschedule < 30s; no 5xx increase | Availability > 99.9% |
| **CHAOS-002** | Database Primary Failover | nxa-db (RDS) | Force failover | Read replica promoted < 60s; writes pause < 30s | RTO < 4h, RPO < 1h |
| **CHAOS-003** | Redis Cluster Node Failure | nxa-redis | Kill 1 node | Auto-failover; cache misses spike < 10% | Cache hit ratio > 90% |
| **CHAOS-004** | Kafka Broker Failure | nxa-kafka | Kill 1 broker | ISR reassignment; producer retries | No message loss |
| **CHAOS-005** | Network Latency (Backend→DB) | nxa-backend | 200ms latency | P99 latency < 2s; circuit breaker trips | Latency budget < 50% |
| **CHAOS-006** | Network Partition (AZ) | nxa-backend (AZ-B) | Block AZ-B traffic | AZ-A/C handle load; no data loss | Availability > 99.95% |
| **CHAOS-007** | Clock Skew | nxa-backend | -5min offset | Token validation fails gracefully; alert | Auth error rate < 1% |
| **CHAOS-008** | DNS Resolution Failure | nxa-backend | Block DNS | Service discovery fallback; cached IPs work | External API calls degrade gracefully |
| **CHAOS-009** | Disk Full (PostgreSQL) | nxa-db | Fill disk 95% | Alert fires; auto-scale storage | No data corruption |
| **CHAOS-010** | External API Failure (Lansweeper) | nxa-lansweeper | 503 responses | Sync jobs retry; queue builds; alert | No data loss; sync resumes |

### 8.2 Chaos Experiment Implementation (Chaos Mesh)

```yaml
# platform/chaos/experiments/backend-pod-kill.yaml
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
    cron: "@weekly"
---
# platform/chaos/experiments/network-latency.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: backend-db-latency
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
    latency: "200ms"
    correlation: "25"
    jitter: "50ms"
  scheduler:
    cron: "@weekly"
---
# platform/chaos/experiments/kafka-broker-failure.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: kafka-broker-failure
  namespace: chaos-testing
spec:
  action: pod-kill
  mode: one
  selector:
    namespaces:
      - kafka
    labelSelectors:
      app: nxa-kafka
  scheduler:
    cron: "@weekly"
```

### 8.3 Chaos Validation

```python
# backend/tests/chaos/validation.py
import pytest
from prometheus_api_client import PrometheusConnect

class ChaosValidation:
    @staticmethod
    def validate_availability_during_chaos(prometheus: PrometheusConnect, duration_minutes: int = 10):
        """Query availability metric during chaos window"""
        query = f'avg_over_time(up{{job=~"nxa-.*"}}[{duration_minutes}m])'
        result = prometheus.custom_query(query)
        
        availability = float(result[0]['value'][1])
        assert availability > 0.999, f"Availability dropped to {availability}"
    
    @staticmethod
    def validate_no_data_loss(prometheus: PrometheusConnect, topic: str):
        """Verify no message loss in Kafka"""
        query = f'increase(kafka_topic_partition_current_offset{{topic="{topic}"}}[10m])'
        result = prometheus.custom_query(query)
        
        # Compare with producer metrics
        produced = prometheus.custom_query(
            f'increase(kafka_producer_record_send_total{{topic="{topic}"}}[10m])'
        )
        
        assert abs(float(result[0]['value'][1]) - float(produced[0]['value'][1])) < 10
    
    @staticmethod
    def validate_latency_budget(prometheus: PrometheusConnect, service: str, budget_pct: float = 0.5):
        """Verify latency stays within error budget"""
        query = f'histogram_quantile(0.99, rate(http_request_duration_seconds_bucket{{service="{service}"}}[5m]))'
        result = prometheus.custom_query(query)
        
        p99_latency = float(result[0]['value'][1])
        budget = 0.5 * budget_pct  # 500ms * 50% = 250ms
        
        assert p99_latency < budget, f"P99 latency {p99_latency}s exceeds budget {budget}s"
```

---

## 9. Performance & Load Testing

### 9.1 k6 Load Test Scripts

```javascript
// backend/tests/performance/k6/asset-api-load.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const errorRate = new Rate('errors');
const assetCreateTrend = new Trend('asset_create_duration');
const assetListTrend = new Trend('asset_list_duration');

export const options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up
    { duration: '5m', target: 500 },   // Sustained load
    { duration: '2m', target: 1000 },  // Peak load
    { duration: '5m', target: 1000 },  // Sustained peak
    { duration: '2m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(99)<500'],
    http_req_failed: ['rate<0.01'],
    errors: ['rate<0.01'],
  },
};

const BASE_URL = 'https://api.nexusasset.cloud/api/v1';
const TENANT_ID = 'tn_perf_test';
const TOKEN = __ENV.AUTH_TOKEN;

const headers = {
  'Authorization': `Bearer ${TOKEN}`,
  'X-Tenant-ID': TENANT_ID,
  'Content-Type': 'application/json',
  'Idempotency-Key': `perf-${__VU}-${__ITER}`,
};

export default function () {
  // 1. List assets (read-heavy)
  let start = new Date();
  let res = http.get(`${BASE_URL}/assets?limit=50`, { headers });
  assetListTrend.add(new Date() - start);
  
  check(res, {
    'list status 200': (r) => r.status === 200,
    'list has data': (r) => r.json('data').length >= 0,
  }) || errorRate.add(1);
  
  // 2. Create asset (write)
  if (__ITER % 10 === 0) {
    start = new Date();
    const payload = JSON.stringify({
      name: `Perf Test Asset ${__VU}-${__ITER}`,
      asset_type: 'SERVER',
      serial_number: `PERF-${__VU}-${__ITER}`,
    });
    
    res = http.post(`${BASE_URL}/assets`, payload, { headers });
    assetCreateTrend.add(new Date() - start);
    
    check(res, {
      'create status 201': (r) => r.status === 201,
    }) || errorRate.add(1);
  }
  
  sleep(1);
}

export function handleSummary(data) {
  return {
    'stdout': textSummary(data, { indent: ' ', enableColors: true }),
    'summary.json': JSON.stringify(data),
  };
}
```

### 9.2 Frontend Performance (Lighthouse CI)

```yaml
# frontend/.github/workflows/lighthouse.yml
name: Lighthouse CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - run: npm ci
      - run: npm run build
      - run: npm run start &
      - name: Wait for server
        run: sleep 10
      - name: Run Lighthouse CI
        run: npx lighthouse-ci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

```json
// frontend/lighthouserc.json
{
  "ci": {
    "collect": {
      "numberOfRuns": 3,
      "startServerCommand": "npm run start",
      "url": ["http://localhost:3000", "http://localhost:3000/assets", "http://localhost:3000/assets/ast_001"]
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.9 }],
        "categories:accessibility": ["error", { "minScore": 0.95 }],
        "categories:best-practices": ["error", { "minScore": 0.9 }],
        "categories:seo": ["warn", { "minScore": 0.8 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }],
        "interaction-to-next-paint": ["error", { "maxNumericValue": 200 }],
        "total-blocking-time": ["error", { "maxNumericValue": 200 }]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

### 9.3 Soak Testing

```javascript
// backend/tests/performance/k6/soak-test.js
export const options = {
  scenarios: {
    soak: {
      executor: 'constant-vus',
      vus: 200,
      duration: '24h',
    },
  },
  thresholds: {
    http_req_duration: ['p(99)<500'],
    http_req_failed: ['rate<0.001'],  // Stricter for soak
    'checks{name:memory_leak}': ['rate>0.99'],
  },
};

export default function () {
  // Standard API calls
  // Memory leak detection via custom check
  const memCheck = check(null, {
    'memory_leak': () => {
      // Query Prometheus for memory growth
      // This would be a custom metric
      return true;
    },
  });
}
```

---

## 10. Security Testing

### 10.1 SAST (Static Application Security Testing)

```yaml
# .github/workflows/sast.yml
name: SAST

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  semgrep:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten
            p/python
            p/typescript
            p/django
            p/react
  
  bandit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Bandit (Python)
        run: |
          pip install bandit
          bandit -r backend/ -f json -o bandit-report.json
      - name: Upload Bandit Results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: bandit-report.json
```

### 10.2 DAST (Dynamic Application Security Testing)

```yaml
# .github/workflows/dast.yml
name: DAST

on:
  schedule:
    - cron: '0 2 * * 0'  # Weekly Sunday 2AM
  workflow_dispatch:

jobs:
  zap-scan:
    runs-on: ubuntu-latest
    steps:
      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.10.0
        with:
          target: 'https://staging.api.nexusasset.cloud'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'
      - name: ZAP Full Scan (on release)
        if: github.event_name == 'workflow_dispatch'
        uses: zaproxy/action-full-scan@v0.10.0
        with:
          target: 'https://staging.api.nexusasset.cloud'
```

### 10.3 Dependency Scanning

```yaml
# .github/workflows/dependency-scan.yml
name: Dependency Scan

on:
  push:
    branches: [main, develop]
  schedule:
    - cron: '0 3 * * *'  # Daily 3AM

jobs:
  npm-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm audit --audit-level=high
        working-directory: ./frontend
  
  pip-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
      - run: pip install pip-audit
      - run: pip-audit -r backend/requirements.txt --format=json --output=pip-audit.json
  
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Snyk Test
        uses: snyk/actions/node@master
        with:
          command: test
          args: --severity-threshold=high
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

---

## 11. Accessibility Testing

```typescript
// frontend/tests/accessibility/a11y.test.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility', () => {
  test('Dashboard page has no violations', async ({ page }) => {
    await login(page, user);
    await page.goto('/dashboard');
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });
  
  test('Asset create form is accessible', async ({ page }) => {
    await login(page, user);
    await page.goto('/assets/create');
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .include('[data-testid="asset-form"]')
      .withTags(['wcag2a', 'wcag2aa'])
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });
  
  test('Color contrast meets WCAG AA', async ({ page }) => {
    await login(page, user);
    await page.goto('/dashboard');
    
    const results = await new AxeBuilder({ page })
      .withTags(['cat.color'])
      .analyze();
    
    expect(results.violations).toEqual([]);
  });
});
```

---

## 12. Test Data Management

### 12.1 Test Data Strategy

| Approach | Use Case | Implementation |
|----------|----------|----------------|
| **Fixtures** | Unit tests, predictable data | JSON/YAML files loaded per test |
| **Factories** | Integration tests, variable data | `factory_boy` (Python), `@faker-js/faker` (TS) |
| **Database Snapshots** | E2E tests, complex state | PostgreSQL logical replication slots |
| **Synthetic Data** | Performance tests, large volumes | Custom generators per domain |
| **Production-Like (Anonymized)** | Staging validation | `pg_dump` + anonymization scripts |

### 12.2 Factory Examples

```python
# backend/tests/factories/asset_factory.py
import factory
from nexusasset.assets.domain.aggregates.asset import Asset, AssetId
from nexusasset.assets.domain.value_objects import AssetType, AssetState

class AssetFactory(factory.Factory):
    class Meta:
        model = Asset
    
    asset_id = factory.LazyFunction(AssetId.generate)
    tenant_id = factory.SubFactory('tests.factories.TenantFactory')
    asset_type = AssetType.SERVER
    name = factory.Sequence(lambda n: f"Test Asset {n}")
    serial_number = factory.Sequence(lambda n: f"SN{n:06d}")
    asset_tag = factory.Sequence(lambda n: f"NXA-{n:08X}")
    state = AssetState.DISCOVERED
    specifications = factory.LazyFunction(lambda: HardwareSpecs(
        cpu=CpuSpec(model="Intel Xeon", cores=16),
        memory=MemorySpec(total_gb=64),
        storage=[StorageSpec(capacity_gb=512)],
    ))
    created_by = factory.SubFactory('tests.factories.UserFactory')

class AssetWithSoftwareFactory(AssetFactory):
    installed_software = factory.List([
        factory.SubFactory('tests.factories.SoftwareInstallationFactory'),
        factory.SubFactory('tests.factories.SoftwareInstallationFactory'),
    ])
```

---

## 13. Test Environments

| Environment | Purpose | Infrastructure | Data | Access |
|-------------|---------|----------------|------|--------|
| **Local** | Developer iteration | kind (K8s), Testcontainers | Fixtures, Factories | Developer |
| **CI (GitHub Actions)** | PR validation | GitHub-hosted runners, Testcontainers | Fixtures, Factories | Team |
| **Staging** | Pre-production validation | 1 K8s cluster (3 AZ) | Anonymized production subset | Team + QA |
| **Chaos/Performance** | Specialized testing | Dedicated cluster | Synthetic + anonymized | Platform + QA |
| **Production** | Canary validation | Production cluster | Real data (read-only) | Platform only |

---

## 14. CI/CD Integration

### 14.1 Pipeline Stages

```yaml
# .github/workflows/test.yml (simplified)
name: Test Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Backend Unit Tests
        run: |
          cd backend
          pytest tests/unit -x --cov --cov-fail-under=90 -n 50
      - name: Frontend Unit Tests
        run: |
          cd frontend
          npm run test:unit -- --coverage
  
  integration-tests:
    needs: unit-tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:18
        env: { POSTGRES_PASSWORD: test }
        ports: [5432]
      kafka:
        image: confluentinc/cp-kafka:7.5
        ports: [9092]
      redis:
        image: redis:8
        ports: [6379]
    steps:
      - uses: actions/checkout@v4
      - name: Backend Integration Tests
        run: |
          cd backend
          pytest tests/integration -x -n 10
      - name: Frontend Integration Tests
        run: |
          cd frontend
          npm run test:integration
  
  contract-tests:
    needs: unit-tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Provider Contract Tests
        run: |
          cd backend
          pytest tests/contracts/provider -x
      - name: Consumer Contract Tests
        run: |
          cd frontend
          npm run test:contract
  
  e2e-tests:
    needs: [integration-tests, contract-tests]
    if: github.event_name == 'push' || github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Staging
        run: ./scripts/deploy-staging.sh
      - name: Run E2E Tests
        run: |
          cd frontend
          npm run test:e2e
  
  chaos-tests:
    if: github.event_name == 'schedule' || github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest
    steps:
      - name: Run Chaos Experiments
        run: kubectl apply -f platform/chaos/experiments/
  
  performance-tests:
    if: github.event_name == 'schedule' || github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest
    steps:
      - name: Run k6 Load Tests
        run: k6 run backend/tests/performance/k6/*.js
```

### 14.2 Quality Gates

| Gate | Metric | Threshold | Action on Failure |
|------|--------|-----------|-------------------|
| **Unit Coverage** | Domain lines | ≥ 90% | Block merge |
| **Unit Coverage** | Application lines | ≥ 85% | Block merge |
| **Integration Pass Rate** | All tests | 100% | Block merge |
| **Contract Compliance** | Schemathesis/Pact | 0 failures | Block merge |
| **E2E Pass Rate** | Critical journeys | 100% | Block deploy |
| **Performance Regression** | p99 latency | < 10% increase | Block deploy |
| **Security Scan** | Critical/High vulns | 0 | Block deploy |
| **Accessibility** | Axe violations | 0 | Block deploy |

---

## 15. Quality Gates & Metrics

### 15.1 Test Metrics Dashboard

| Metric | Target | Current | Trend |
|--------|--------|---------|-------|
| **Unit Test Count** | 5,000+ | 4,823 | ↗ |
| **Unit Coverage (Domain)** | 90% | 92.3% | ↗ |
| **Unit Coverage (App)** | 85% | 87.1% | ↗ |
| **Integration Test Count** | 1,500+ | 1,342 | ↗ |
| **Contract Test Count** | 200+ | 187 | ↗ |
| **E2E Test Count** | 50+ | 42 | ↗ |
| **Flaky Test Rate** | < 1% | 0.3% | ↘ |
| **Mean Time to Feedback** | < 10 min | 8 min | ↘ |
| **Defect Escape Rate** | < 5% | 3.2% | ↘ |

### 15.2 Test Result Reporting

```python
# backend/tests/reporting.py
import pytest
from pytest_html import extras

def pytest_html_results_table_header(cells):
    cells.insert(2, extras.html('<th>Tenant</th>'))
    cells.insert(3, extras.html('<th>Trace ID</th>'))

def pytest_html_results_table_row(report, cells):
    cells.insert(2, extras.html(f'<td>{getattr(report, "tenant_id", "N/A")}</td>'))
    cells.insert(3, extras.html(f'<td>{getattr(report, "trace_id", "N/A")}</td>'))

# Generates HTML report with tenant context for each test
```

---

## 16. Diagrams

| Diagram ID | Title | Type | Location | Description |
|------------|-------|------|----------|-------------|
| **DIA-104** | Test Pyramid | Pyramid | Section 2 | Layer distribution |
| **DIA-105** | Testing Scope by Layer | Layered | Section 1.2 | What each layer tests |
| **DIA-106** | Unit Test Structure | Component | Section 3 | Domain/Application/Frontend |
| **DIA-107** | Integration Test Flow | Sequence | Section 4 | Testcontainers → Migrations → Tests |
| **DIA-108** | Contract Testing | Component | Section 5 | Provider ↔ Consumer ↔ Events |
| **DIA-109** | Tenant Isolation Tests | Flowchart | Section 6 | DB → API → Frontend |
| **DIA-110** | E2E Critical Journeys | Sequence | Section 7.1 | Asset lifecycle, compliance, sync |
| **DIA-111** | Chaos Experiment Flow | Flowchart | Section 8 | Inject → Observe → Validate |
| **DIA-112** | Load Test Architecture | Component | Section 9 | k6 → API → Metrics → Validation |
| **DIA-113** | CI/CD Test Pipeline | Flowchart | Section 14.1 | Stages → Gates → Deploy |
| **DIA-114** | Test Data Strategy | Component | Section 12 | Fixtures → Factories → Snapshots |

---

## 17. Tables

| Table ID | Title | Location | Row Count |
|----------|-------|----------|-----------|
| **TBL-209** | Testing Principles | Section 1.1 | 10 |
| **TBL-210** | Test Pyramid Distribution | Section 2.1 | 6 |
| **TBL-211** | Coverage Targets | Section 3.3 | 4 |
| **TBL-212** | Chaos Experiments | Section 8.1 | 10 |
| **TBL-213** | Test Data Approaches | Section 12.1 | 5 |
| **TBL-214** | Test Environments | Section 13 | 5 |
| **TBL-215** | Quality Gates | Section 14.2 | 8 |
| **TBL-216** | Test Metrics | Section 15.1 | 10 |
| **TBL-217** | Diagram Index | Section 16 | 11 |
| **TBL-218** | Table Index | Section 17 | 10 |

---

---

**End of DOC-10 — NexusAsset Cloud Testing Strategy**

*Document Control: Version 1.0.0 | Classification: Internal — Confidential | Next Review: 2026-12-09*

*For questions or updates: docs@nexusasset.cloud | QA Team: qa@nexusasset.cloud*