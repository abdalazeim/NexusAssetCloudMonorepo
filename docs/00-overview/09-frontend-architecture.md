╔═════════════════════════════════════════════════════════════════════════════════════════════════╗
║ Document ID: DOC-09                                                                            ║
║ Title: Frontend Architecture — Next.js 16, React 19, BFF, Micro-Frontends                      ║
║ Version: 1.0.0                                                                                 ║
║ Status: ✓ Approved                                                                             ║
║ Classification: Internal — Confidential                                                        ║
║ Authors: NexusAsset Cloud Frontend Team                                                        ║
║ Reviewers: Backend, UX/UI, Security, Platform, Accessibility                                   ║
║ Approved By: CTO / VP Engineering / Frontend Architect                                         ║
║ Date: 2026-11-09                                                                               ║
║ Supersedes: —                                                                                  ║
║ Related: DOC-00, DOC-01, DOC-02, DOC-03, DOC-04, DOC-05, DOC-06, DOC-07, DOC-10, DOC-13       ║
║ Traceability: REQ-FN-### ← UC-###, FEAT-### → Component, ADR-### → Frontend Decisions          ║
╚════════════════════════════════════════════════════════════════════════════════════════════════╝

# ═══════════════════════════════════════════════════════════════
# NexusAsset Cloud — Frontend Architecture (DOC-09)
# ═══════════════════════════════════════════════════════════════

> **Executive Summary** — This document defines the frontend architecture for NexusAsset Cloud using **Next.js 16** (App Router), **React 19** (Server Components), **TypeScript 5.5**, and **Tailwind CSS 3.4**. The architecture follows a **BFF (Backend-for-Frontend)** pattern with **micro-frontend** capabilities for tenant-specific customizations. Key principles: **Server-First Rendering** (RSC), **Progressive Enhancement**, **Accessibility First** (WCAG 2.1 AA), **Performance Budgets**, and **Type-Safe API Integration** via generated SDK. The frontend is deployed as a standalone Next.js application on Kubernetes with edge caching via Cloudflare.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Tech Stack & Versions](#2-tech-stack--versions)
3. [Project Structure](#3-project-structure)
4. [Rendering Strategy](#4-rendering-strategy)
5. [State Management](#5-state-management)
6. [BFF Integration](#6-bff-integration)
7. [Micro-Frontend Architecture](#7-micro-frontend-architecture)
8. [Component Architecture](#8-component-architecture)
9. [Routing & Navigation](#9-routing--navigation)
10. [Authentication & Authorization](#10-authentication--authorization)
11. [Internationalization (i18n)](#11-internationalization-i18n)
12. [Accessibility (a11y)](#12-accessibility-a11y)
13. [Performance Optimization](#13-performance-optimization)
14. [Testing Strategy](#14-testing-strategy)
15. [Deployment & CI/CD](#15-deployment--cicd)
16. [Security Considerations](#16-security-considerations)
17. [Diagrams](#17-diagrams)
18. [Tables](#18-tables)

---

## 1. Architecture Overview

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              FRONTEND ARCHITECTURE                                                  │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                              EDGE (Cloudflare CDN)                                          │   │
│  │  • Static asset caching (immutable, 1 year)                                                 │   │
│  │  • HTML caching (stale-while-revalidate, 60s)                                              │   │
│  │  • WAF, Bot Management, DDoS Protection                                                    │   │
│  │  • Geo-routing to nearest region                                                            │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                           │                                                         │
│                                           ▼                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                        NEXT.JS 16 APPLICATION (Kubernetes)                                  │   │
│  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │   │
│  │  │                        APP ROUTER (Server Components by Default)                    │   │   │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │   │   │
│  │  │  │ (auth)      │  │ (dashboard) │  │ (assets)    │  │ (settings)  │  ...         │   │   │
│  │  │  │ layout.tsx  │  │ layout.tsx  │  │ layout.tsx  │  │ layout.tsx  │              │   │   │
│  │  │  │ page.tsx    │  │ page.tsx    │  │ page.tsx    │  │ page.tsx    │              │   │   │
│  │  │  │ loading.tsx │  │ loading.tsx │  │ loading.tsx │  │ loading.tsx │              │   │   │
│  │  │  │ error.tsx   │  │ error.tsx   │  │ error.tsx   │  │ error.tsx   │              │   │   │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘              │   │   │
│  │  └─────────────────────────────────────────────────────────────────────────────────────┘   │   │
│  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │   │
│  │  │                        BFF LAYER (Route Handlers / API Routes)                      │   │   │
│  │  │  /api/bff/* → Aggregates backend calls, transforms for UI                          │   │   │
│  │  │  /api/auth/* → NextAuth.js (JWT, SSO, MFA)                                         │   │   │
│  │  │  /api/webhooks/* → Webhook receivers                                                │   │   │
│  │  └─────────────────────────────────────────────────────────────────────────────────────┘   │   │
│  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐   │   │
│  │  │                        SHARED LIBRARIES                                             │   │   │
│  │  │  @nexusasset/ui (Component Library)   @nexusasset/sdk (API Client)                 │   │   │
│  │  │  @nexusasset/hooks (React Hooks)      @nexusasset/config (Runtime Config)          │   │   │
│  │  └─────────────────────────────────────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                           │                                                         │
│                    ┌──────────────────────┼──────────────────────┐                              │
│                    ▼                      ▼                      ▼                              │
│           ┌───────────────┐       ┌───────────────┐       ┌───────────────┐                   │
│           │  BACKEND API  │       │  BFF ENDPOINTS│       │  EXTERNAL     │                   │
│           │  (nxa-backend)│       │  (nxa-frontend)│       │  (Auth, CDN)  │                   │
│           │  /api/v1/*    │       │  /api/bff/*   │       │               │                   │
│           └───────────────┘       └───────────────┘       └───────────────┘                   │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Design Principles

| Principle ID | Principle | Implementation |
|--------------|-----------|----------------|
| **FE-PRIN-001** | **Server-First** | Default to React Server Components; Client Components only for interactivity |
| **FE-PRIN-002** | **Type Safety End-to-End** | OpenAPI → TypeScript types → SDK → Component props |
| **FE-PRIN-003** | **Progressive Enhancement** | Core functionality works without JS; enhanced with React |
| **FE-PRIN-004** | **Accessibility by Default** | WCAG 2.1 AA; semantic HTML; ARIA only when necessary |
| **FE-PRIN-005** | **Performance Budgets** | LCP < 2.5s, CLS < 0.1, TBT < 200ms, Bundle < 200KB gzipped |
| **FE-PRIN-006** | **Tenant-Aware** | Runtime config per tenant; theming; feature flags |
| **FE-PRIN-007** | **Micro-Frontend Ready** | Module Federation for tenant-specific extensions |
| **FE-PRIN-008** | **Observability** | RUM (Real User Monitoring), Core Web Vitals, Error tracking |

---

## 2. Tech Stack & Versions

| Category | Technology | Version | Purpose |
|----------|------------|---------|---------|
| **Framework** | Next.js | 16.x (App Router) | Full-stack React framework |
| **UI Library** | React | 19.x | Server Components, Actions, Suspense |
| **Language** | TypeScript | 5.5 | Static typing |
| **Styling** | Tailwind CSS | 3.4 | Utility-first CSS |
| **UI Components** | Radix UI / shadcn/ui | Latest | Accessible primitives |
| **Forms** | React Hook Form + Zod | 7.5 / 3.2 | Validation, type-safe forms |
| **State** | TanStack Query (React Query) | 5.x | Server state, caching |
| **Client State** | Zustand | 4.x | Global client state |
| **Animation** | Framer Motion | 11.x | Transitions, animations |
| **Charts** | Recharts / Tremor | 2.x / 3.x | Data visualization |
| **Tables** | TanStack Table | 8.x | Headless table logic |
| **Date** | date-fns | 3.x | Date formatting |
| **Icons** | Lucide React | 0.4.x | Icon system |
| **Notifications** | Sonner | 1.x | Toast notifications |
| **Auth** | NextAuth.js | 5.x (beta) | Authentication |
| **Testing** | Vitest, Playwright, Testing Library | Latest | Unit, E2E, Component |
| **Linting** | ESLint, Prettier, TypeScript | Latest | Code quality |
| **Build** | Turbopack (Next.js) | Built-in | Fast bundling |

---

## 3. Project Structure

```
frontend/
├── .github/
│   └── workflows/              # CI/CD pipelines
├── public/
│   ├── fonts/                  # Self-hosted fonts (Inter, JetBrains Mono)
│   ├── images/                 # Static images
│   └── manifest.json           # PWA manifest
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (auth)/             # Auth route group
│   │   │   ├── login/
│   │   │   │   ├── page.tsx
│   │   │   │   └── LoginForm.tsx
│   │   │   ├── mfa/
│   │   │   │   └── page.tsx
│   │   │   ├── sso/
│   │   │   │   └── callback/
│   │   │   └── layout.tsx
│   │   ├── (dashboard)/        # Main app route group (protected)
│   │   │   ├── layout.tsx      # Root layout with sidebar, header
│   │   │   ├── page.tsx        # Dashboard overview
│   │   │   ├── assets/
│   │   │   │   ├── page.tsx                    # Asset list
│   │   │   │   ├── [id]/
│   │   │   │   │   ├── page.tsx                # Asset detail
│   │   │   │   │   ├── relationships/
│   │   │   │   │   ├── history/
│   │   │   │   │   └── maintenance/
│   │   │   │   ├── create/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── import/
│   │   │   │   └── bulk-actions/
│   │   │   ├── tenants/
│   │   │   ├── billing/
│   │   │   ├── compliance/
│   │   │   ├── settings/
│   │   │   └── layout.tsx
│   │   ├── api/                # BFF / API Routes
│   │   │   ├── auth/           # NextAuth.js endpoints
│   │   │   │   ├── [...nextauth]/
│   │   │   │   └── route.ts
│   │   │   ├── bff/            # BFF aggregation endpoints
│   │   │   │   ├── assets/
│   │   │   │   │   ├── dashboard/route.ts
│   │   │   │   │   ├── detail/[id]/route.ts
│   │   │   │   │   └── bulk/route.ts
│   │   │   │   ├── tenants/
│   │   │   │   └── compliance/
│   │   │   ├── webhooks/       # Webhook receivers
│   │   │   └── health/         # Health checks
│   │   ├── globals.css         # Global styles, Tailwind imports
│   │   ├── layout.tsx          # Root layout (providers, fonts)
│   │   ├── page.tsx            # Landing page (redirects to dashboard)
│   │   ├── loading.tsx         # Global loading UI
│   │   ├── error.tsx           # Global error UI
│   │   ├── not-found.tsx       # 404 page
│   │   └── robots.txt          # Robots.txt
│   ├── components/             # Shared components
│   │   ├── ui/                 # Base UI components (Button, Input, Table, etc.)
│   │   ├── layout/             # Layout components (Sidebar, Header, Breadcrumbs)
│   │   ├── assets/             # Asset-specific components
│   │   ├── tenants/            # Tenant-specific components
│   │   ├── billing/            # Billing components
│   │   ├── compliance/         # Compliance components
│   │   ├── charts/             # Chart components
│   │   ├── forms/              # Form components
│   │   ├── tables/             # Table components
│   │   └── providers/          # Context providers
│   ├── lib/                    # Utilities & configuration
│   │   ├── sdk/                # Generated API SDK (from OpenAPI)
│   │   ├── hooks/              # Custom React hooks
│   │   ├── utils/              # Utility functions
│   │   ├── constants/          # Application constants
│   │   ├── config/             # Runtime configuration
│   │   ├── auth/               # Auth utilities
│   │   ├── i18n/               # Internationalization
│   │   └── validators/         # Zod schemas
│   ├── styles/                 # Additional styles
│   │   ├── globals.css
│   │   ├── tailwind.css
│   │   └── themes/             # Tenant themes
│   ├── types/                  # TypeScript types
│   │   ├── api/                # Generated from OpenAPI
│   │   ├── domain/             # Domain types (mirror backend)
│   │   └── ui/                 # UI-specific types
│   └── middleware.ts           # Next.js middleware (auth, tenant, i18n)
├── package.json
├── tsconfig.json
├── next.config.js
├── tailwind.config.ts
├── postcss.config.js
├── eslint.config.js
├── prettier.config.js
├── vitest.config.ts
├── playwright.config.ts
├── .env.example
├── .env.local                  # Local overrides (gitignored)
├── dockerfile
├── docker-compose.yml
└── README.md
```

---

## 4. Rendering Strategy

### 4.1 Rendering Decision Matrix

| Page / Component | Strategy | Reason |
|------------------|----------|--------|
| **Landing Page** | Static (ISR) | Public, SEO, infrequent changes |
| **Login / MFA / SSO** | Server Component | Auth forms, no client state needed |
| **Dashboard** | Server Component + Client Islands | Data fetching on server; interactive widgets client-side |
| **Asset List** | Server Component | SEO not needed; server-side filtering/sorting |
| **Asset Detail** | Server Component + Client Components | Initial data server; tabs, actions client-side |
| **Asset Create/Edit** | Client Component | Forms, validation, optimistic updates |
| **Tenant Settings** | Server Component + Client | Settings forms client-side |
| **Compliance Reports** | Server Component (Streaming) | Large data sets, streaming |
| **Settings / Profile** | Client Component | Interactive forms |
| **Real-time Notifications** | Client Component (WebSocket) | Live updates |

### 4.2 Server Component Example (Asset List)

```tsx
// src/app/(dashboard)/assets/page.tsx
import { AssetList } from '@/components/assets/AssetList';
import { getAssetsServer } from '@/lib/sdk/server';
import { AssetListParams } from '@/types/api';

interface PageProps {
  searchParams: Promise<AssetListParams>;
}

export default async function AssetsPage({ searchParams }: PageProps) {
  const params = await searchParams;
  
  // Fetch on server (no client bundle)
  const { data: assets, meta } = await getAssetsServer(params);
  
  return (
    <AssetList
      initialAssets={assets}
      initialMeta={meta}
      searchParams={params}
    />
  );
}
```

### 4.3 Client Component Example (Asset Detail Tabs)

```tsx
// src/components/assets/AssetDetailTabs.tsx
'use client';

import { useState } from 'react';
import { Tabs, TabsList, TabsTrigger, TabsContent } from '@/components/ui/Tabs';
import { AssetRelationships } from './AssetRelationships';
import { AssetHistory } from './AssetHistory';
import { AssetMaintenance } from './AssetMaintenance';
import { AssetCompliance } from './AssetCompliance';

export function AssetDetailTabs({ assetId }: { assetId: string }) {
  const [activeTab, setActiveTab] = useState<'overview' | 'relationships' | 'history' | 'maintenance' | 'compliance'>('overview');
  
  return (
    <Tabs value={activeTab} onValueChange={setActiveTab}>
      <TabsList>
        <TabsTrigger value="overview">Overview</TabsTrigger>
        <TabsTrigger value="relationships">Relationships</TabsTrigger>
        <TabsTrigger value="history">History</TabsTrigger>
        <TabsTrigger value="maintenance">Maintenance</TabsTrigger>
        <TabsTrigger value="compliance">Compliance</TabsTrigger>
      </TabsList>
      
      <TabsContent value="overview">
        {/* Overview content */}
      </TabsContent>
      <TabsContent value="relationships">
        <AssetRelationships assetId={assetId} />
      </TabsContent>
      <TabsContent value="history">
        <AssetHistory assetId={assetId} />
      </TabsContent>
      <TabsContent value="maintenance">
        <AssetMaintenance assetId={assetId} />
      </TabsContent>
      <TabsContent value="compliance">
        <AssetCompliance assetId={assetId} />
      </TabsContent>
    </Tabs>
  );
}
```

---

## 5. State Management

### 5.1 State Layers

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              STATE MANAGEMENT LAYERS                                                │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                        SERVER STATE (TanStack Query)                                        │   │
│  │  • API responses (assets, tenants, billing, compliance)                                    │   │
│  │  • Caching, deduplication, background refetch                                              │   │
│  │  • Optimistic updates for mutations                                                        │   │
│  │  • Persistence: IndexedDB (optional)                                                       │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                           │                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                        CLIENT STATE (Zustand)                                               │   │
│  │  • UI state: sidebar open/closed, modals, toasts                                           │   │
│  │  • User preferences: theme, density, language                                              │   │
│  │  • Feature flags (from nxa_core)                                                           │   │
│  │  • Tenant context (current tenant, available tenants)                                      │   │
│  │  • Persistence: localStorage                                                                 │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                           │                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                        FORM STATE (React Hook Form)                                         │   │
│  │  • Controlled/uncontrolled inputs                                                          │   │
│  │  • Validation (Zod schemas)                                                                │   │
│  │  • Dirty tracking, submit handling                                                         │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                           │                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                        URL STATE (Next.js Search Params)                                    │   │
│  │  • Filters, sorting, pagination                                                            │   │
│  │  • Deep-linkable, shareable                                                                │   │
│  │  • Synced with TanStack Query                                                              │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                           │                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│  │                        SERVER STATE (Next.js Cache)                                         │   │
│  │  • Static/ISR pages                                                                        │   │
│  │  • Route segment cache                                                                     │   │
│  │  • Data cache (fetch with next: { cache: 'force-cache' | 'no-store' })                    │   │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 TanStack Query Configuration

```typescript
// src/lib/query/client.ts
import { QueryClient } from '@tanstack/react-query';
import { persistQueryClient } from '@tanstack/query-sync-storage-persister';
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister';

export function createQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 1000 * 60 * 5,      // 5 minutes
        gcTime: 1000 * 60 * 30,        // 30 minutes
        retry: (failureCount, error) => {
          if (error instanceof ApiError && error.status === 401) return false;
          return failureCount < 3;
        },
        retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
        refetchOnWindowFocus: false,
        refetchOnReconnect: true,
      },
      mutations: {
        retry: 0,
        onError: (error) => {
          if (error instanceof ApiError && error.status === 401) {
            // Trigger auth redirect
            authStore.getState().signOut();
          }
        },
      },
    },
  });
}

// Persistence for offline support
const persister = createSyncStoragePersister({
  storage: window.localStorage,
  throttleTime: 1000 * 60, // 1 minute
});

persistQueryClient({
  queryClient,
  persister,
  maxAge: 1000 * 60 * 60 * 24, // 24 hours
});
```

### 5.3 Zustand Store (Client State)

```typescript
// src/lib/stores/app-store.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { FeatureFlag } from '@/types/api';

interface AppState {
  // UI State
  sidebarOpen: boolean;
  toggleSidebar: () => void;
  setSidebarOpen: (open: boolean) => void;
  
  // User Preferences
  theme: 'light' | 'dark' | 'system';
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
  density: 'comfortable' | 'compact';
  setDensity: (density: 'comfortable' | 'compact') => void;
  language: string;
  setLanguage: (lang: string) => void;
  
  // Tenant Context
  currentTenantId: string | null;
  availableTenants: TenantSummary[];
  setCurrentTenant: (tenantId: string) => void;
  setAvailableTenants: (tenants: TenantSummary[]) => void;
  
  // Feature Flags
  featureFlags: Record<string, FeatureFlag>;
  setFeatureFlags: (flags: Record<string, FeatureFlag>) => void;
  isFeatureEnabled: (key: string) => boolean;
  
  // Notifications
  toasts: Toast[];
  addToast: (toast: Omit<Toast, 'id'>) => string;
  removeToast: (id: string) => void;
}

export const useAppStore = create<AppState>()(
  persist(
    (set, get) => ({
      sidebarOpen: true,
      toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
      setSidebarOpen: (open) => set({ sidebarOpen: open }),
      
      theme: 'system',
      setTheme: (theme) => set({ theme }),
      density: 'comfortable',
      setDensity: (density) => set({ density }),
      language: 'en',
      setLanguage: (language) => set({ language }),
      
      currentTenantId: null,
      availableTenants: [],
      setCurrentTenant: (tenantId) => set({ currentTenantId: tenantId }),
      setAvailableTenants: (tenants) => set({ availableTenants: tenants }),
      
      featureFlags: {},
      setFeatureFlags: (flags) => set({ featureFlags: flags }),
      isFeatureEnabled: (key) => get().featureFlags[key]?.enabled ?? false,
      
      toasts: [],
      addToast: (toast) => {
        const id = crypto.randomUUID();
        set((s) => ({ toasts: [...s.toasts, { ...toast, id }] }));
        return id;
      },
      removeToast: (id) => set((s) => ({ toasts: s.toasts.filter((t) => t.id !== id) })),
    }),
    {
      name: 'nxa-app-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        theme: state.theme,
        density: state.density,
        language: state.language,
        currentTenantId: state.currentTenantId,
      }),
    }
  )
);
```

---

## 6. BFF Integration

### 6.1 BFF Endpoint Examples

```typescript
// src/app/api/bff/assets/dashboard/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSdk } from '@/lib/sdk/server';

export async function GET(request: NextRequest) {
  const sdk = getServerSdk(request);
  const searchParams = request.nextUrl.searchParams;
  
  // Parallel fetch for dashboard data
  const [assets, assetCount, complianceScore, recentActivity] = await Promise.all([
    sdk.assets.list({ limit: 5, sort: '-created_at', state: 'ACTIVE' }),
    sdk.assets.count({ state: 'ACTIVE' }),
    sdk.compliance.getScore(),
    sdk.audit.getRecentActivity({ limit: 10 }),
  ]);
  
  return NextResponse.json({
    assets: assets.data,
    stats: {
      totalAssets: assetCount,
      complianceScore,
      recentActivity: recentActivity.data,
    },
  });
}

// src/app/api/bff/assets/detail/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const sdk = getServerSdk(request);
  
  const [asset, relationships, history, maintenance, compliance] = await Promise.all([
    sdk.assets.get(id, { include: ['specifications', 'owner', 'location'] }),
    sdk.assets.getRelationships(id),
    sdk.assets.getHistory(id, { limit: 50 }),
    sdk.maintenance.list({ asset_id: id }),
    sdk.compliance.getAssetFindings(id),
  ]);
  
  return NextResponse.json({
    asset: asset.data,
    relationships: relationships.data,
    history: history.data,
    maintenance: maintenance.data,
    compliance: compliance.data,
  });
}
```

### 6.2 SDK Usage in Server Components

```typescript
// src/lib/sdk/server.ts
import { NexusAssetClient } from '@nexusasset/sdk';
import { getAuthToken } from '@/lib/auth/server';

export function getServerSdk(request: Request) {
  const token = getAuthToken(request);
  const tenantId = getTenantId(request);
  
  return new NexusAssetClient({
    baseUrl: process.env.NEXT_PUBLIC_API_URL,
    apiKey: token,
    tenantId,
    // Server-side fetch options
    fetchOptions: {
      headers: {
        'X-Request-ID': crypto.randomUUID(),
        'X-Tenant-ID': tenantId || '',
      },
    },
  });
}
```

### 6.3 Client-Side SDK (TanStack Query Integration)

```typescript
// src/lib/sdk/client.ts
import { NexusAssetClient } from '@nexusasset/sdk';
import { useAppStore } from '@/lib/stores/app-store';

let clientInstance: NexusAssetClient | null = null;

export function getClientSdk() {
  if (!clientInstance) {
    const token = useAppStore.getState().authToken;
    const tenantId = useAppStore.getState().currentTenantId;
    
    clientInstance = new NexusAssetClient({
      baseUrl: process.env.NEXT_PUBLIC_API_URL,
      apiKey: token,
      tenantId,
    });
  }
  return clientInstance;
}

// React Query hooks
// src/lib/hooks/useAssets.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { getClientSdk } from '@/lib/sdk/client';

export function useAssets(params: AssetListParams) {
  const sdk = getClientSdk();
  
  return useQuery({
    queryKey: ['assets', params],
    queryFn: () => sdk.assets.list(params),
    placeholderData: (previous) => previous, // Keep previous data while loading
  });
}

export function useCreateAsset() {
  const queryClient = useQueryClient();
  const sdk = getClientSdk();
  
  return useMutation({
    mutationFn: (data: CreateAssetInput) => sdk.assets.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['assets'] });
    },
  });
}
```

---

## 7. Micro-Frontend Architecture

### 7.1 Module Federation Configuration

```javascript
// next.config.js
const withModuleFederation = require('@module-federation/nextjs-mf');

module.exports = withModuleFederation({
  name: 'nxa_frontend',
  exposes: {
    './AssetDashboard': './src/components/assets/AssetDashboard',
    './ComplianceWidget': './src/components/compliance/ComplianceWidget',
    './BillingSummary': './src/components/billing/BillingSummary',
  },
  shared: {
    react: { singleton: true, eager: true },
    'react-dom': { singleton: true, eager: true },
    '@tanstack/react-query': { singleton: true },
    zustand: { singleton: true },
  },
  // Remote tenant-specific MFEs
  remotes: {
    // Loaded dynamically per tenant
  },
});
```

### 7.2 Dynamic Tenant Micro-Frontend Loading

```tsx
// src/components/MicroFrontendLoader.tsx
'use client';

import { Suspense, lazy, useEffect, useState } from 'react';
import { ModuleFederation } from '@module-federation/nextjs-mf';

interface MicroFrontendProps {
  name: string; // e.g., 'tenant-custom-dashboard'
  fallback?: React.ReactNode;
}

export function MicroFrontendLoader({ name, fallback = <div>Loading...</div> }: MicroFrontendProps) {
  const [Component, setComponent] = useState<React.ComponentType<any> | null>(null);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    // Load remote entry from tenant config
    const remoteUrl = `https://mfe.nexusasset.cloud/${name}/remoteEntry.js`;
    
    ModuleFederation.loadRemote(remoteUrl)
      .then((module) => {
        setComponent(() => module.default);
      })
      .catch((err) => {
        console.error(`Failed to load MFE ${name}:`, err);
        setError(err);
      });
  }, [name]);
  
  if (error) return <div>Failed to load {name}</div>;
  if (!Component) return <>{fallback}</>;
  
  return <Component />;
}
```

### 7.3 Tenant MFE Manifest

```json
// Tenant-specific MFE manifest (served from CDN)
{
  "tenant_id": "tn_abc123",
  "micro_frontends": [
    {
      "name": "custom-dashboard",
      "version": "1.2.0",
      "entry": "https://cdn.nexusasset.cloud/mfe/tn_abc123/custom-dashboard/1.2.0/remoteEntry.js",
      "slots": ["dashboard-widgets", "asset-detail-tabs"]
    },
    {
      "name": "custom-compliance",
      "version": "1.0.0",
      "entry": "https://cdn.nexusasset.cloud/mfe/tn_abc123/custom-compliance/1.0.0/remoteEntry.js",
      "slots": ["compliance-dashboard"]
    }
  ]
}
```

---

## 8. Component Architecture

### 8.1 Component Hierarchy

```
src/components/
├── ui/                      # Base UI primitives (design system)
│   ├── Button.tsx
│   ├── Input.tsx
│   ├── Select.tsx
│   ├── Table.tsx
│   ├── Tabs.tsx
│   ├── Modal.tsx
│   ├── Tooltip.tsx
│   ├── Avatar.tsx
│   ├── Badge.tsx
│   ├── Card.tsx
│   ├── Dropdown.tsx
│   ├── Toaster.tsx
│   └── index.ts
├── layout/                  # Layout components
│   ├── AppLayout.tsx        # Main layout with sidebar/header
│   ├── Sidebar.tsx
│   ├── Header.tsx
│   ├── Breadcrumbs.tsx
│   ├── PageHeader.tsx
│   └── Footer.tsx
├── assets/                  # Asset domain components
│   ├── AssetList.tsx
│   ├── AssetCard.tsx
│   ├── AssetDetail.tsx
│   ├── AssetForm.tsx
│   ├── AssetImport.tsx
│   ├── AssetRelationships.tsx
│   ├── AssetHistory.tsx
│   ├── AssetMaintenance.tsx
│   ├── AssetCompliance.tsx
│   └── AssetBulkActions.tsx
├── tenants/                 # Tenant domain components
│   ├── TenantSwitcher.tsx
│   ├── TenantSettings.tsx
│   ├── TenantUsage.tsx
│   └── TenantUsers.tsx
├── billing/                 # Billing domain components
│   ├── SubscriptionCard.tsx
│   ├── InvoiceTable.tsx
│   ├── BillingSummary.tsx
│   └── PlanComparison.tsx
├── compliance/              # Compliance domain components
│   ├── ComplianceDashboard.tsx
│   ├── FrameworkScorecard.tsx
│   ├── FindingTable.tsx
│   ├── EvidenceViewer.tsx
│   └── RiskHeatmap.tsx
├── charts/                  # Chart components
│   ├── AssetTypeChart.tsx
│   ├── ComplianceTrendChart.tsx
│   ├── SyncThroughputChart.tsx
│   └── CostAllocationChart.tsx
├── forms/                   # Form components
│   ├── FormField.tsx
│   ├── FormSection.tsx
│   ├── ArrayField.tsx
│   └── Wizard.tsx
├── tables/                  # Table components
│   ├── DataTable.tsx
│   ├── ColumnDef.tsx
│   ├── Toolbar.tsx
│   └── Pagination.tsx
└── providers/               # Context providers
    ├── QueryProvider.tsx
    ├── AuthProvider.tsx
    ├── ThemeProvider.tsx
    ├── FeatureFlagProvider.tsx
    └── ToasterProvider.tsx
```

### 8.2 Component Design Standards

| Standard | Rule |
|----------|------|
| **Props Interface** | Every component has explicit `Props` interface; exported |
| **Default Props** | Use TypeScript defaults, not `defaultProps` |
| **Forward Ref** | Use `forwardRef` for DOM elements |
| **Accessibility** | Semantic HTML; ARIA only when needed; keyboard navigation |
| **Styling** | Tailwind classes; `cn()` utility for conditional classes |
| **Testing** | Each component has `.test.tsx` with RTL tests |
| **Storybook** | Stories for all UI primitives; design system documentation |
| **Bundle Size** | Tree-shakable exports; no barrel file anti-patterns |

---

## 9. Routing & Navigation

### 9.1 Route Structure

```
/
├── (auth)/
│   ├── login/
│   ├── mfa/
│   ├── sso/callback/
│   └── register/
├── (dashboard)/
│   ├── page.tsx                    # Dashboard overview
│   ├── assets/
│   │   ├── page.tsx                # Asset list
│   │   ├── create/
│   │   ├── import/
│   │   └── [id]/
│   │       ├── page.tsx            # Asset detail (overview)
│   │       ├── relationships/
│   │       ├── history/
│   │       ├── maintenance/
│   │       └── compliance/
│   ├── tenants/
│   │   ├── page.tsx
│   │   ├── [id]/
│   │   └── settings/
│   ├── billing/
│   │   ├── page.tsx
│   │   ├── subscriptions/
│   │   ├── invoices/
│   │   └── contracts/
│   ├── compliance/
│   │   ├── page.tsx
│   │   ├── frameworks/
│   │   ├── assessments/
│   │   ├── findings/
│   │   └── risks/
│   ├── lansweeper/
│   │   ├── page.tsx
│   │   ├── sites/
│   │   └── conflicts/
│   └── settings/
│       ├── page.tsx
│       ├── profile/
│       ├── api-keys/
│       ├── notifications/
│       └── appearance/
```

### 9.2 Middleware (Auth, Tenant, i18n)

```typescript
// src/middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { getToken } from 'next-auth/jwt';
import { getTenantFromRequest } from '@/lib/auth/tenant';

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  
  // Skip static files, API routes (handled separately)
  if (pathname.startsWith('/_next') || pathname.startsWith('/api')) {
    return NextResponse.next();
  }
  
  // Resolve tenant context
  const tenantContext = getTenantFromRequest(request);
  const response = NextResponse.next();
  
  // Set tenant headers for downstream
  if (tenantContext.tenantId) {
    response.headers.set('X-Tenant-ID', tenantContext.tenantId);
  }
  if (tenantContext.locale) {
    response.headers.set('X-Locale', tenantContext.locale);
  }
  
  // Auth protection for dashboard routes
  if (pathname.startsWith('/dashboard') || pathname.startsWith('/assets') || pathname.startsWith('/tenants')) {
    const token = await getToken({ req: request, secret: process.env.NEXTAUTH_SECRET });
    
    if (!token) {
      const loginUrl = new URL('/login', request.url);
      loginUrl.searchParams.set('callbackUrl', pathname);
      return NextResponse.redirect(loginUrl);
    }
    
    // Check tenant access
    if (tenantContext.tenantId && !token.tenants?.includes(tenantContext.tenantId)) {
      return NextResponse.redirect(new URL('/tenants/select', request.url));
    }
  }
  
  return response;
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|robots.txt|sitemap.xml).*)',
  ],
}
```

---

## 10. Authentication & Authorization

### 10.1 NextAuth.js Configuration

```typescript
// src/lib/auth/nextauth.ts
import NextAuth from 'next-auth';
import { authConfig } from './auth-config';

export const { handlers, auth, signIn, signOut } = NextAuth(authConfig);

// src/lib/auth/auth-config.ts
import { NexusAssetAdapter } from './nexusasset-adapter';
import Credentials from 'next-auth/providers/credentials';
import { z } from 'zod';

export const authConfig = {
  adapter: NexusAssetAdapter,
  providers: [
    Credentials({
      name: 'Email/Password',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
        mfa_code: { label: 'MFA Code', type: 'text' },
      },
      authorize: async (credentials) => {
        const parsed = z.object({
          email: z.string().email(),
          password: z.string().min(8),
          mfa_code: z.string().length(6).optional(),
        }).safeParse(credentials);
        
        if (!parsed.success) return null;
        
        const sdk = getServerSdk();
        const result = await sdk.auth.login({
          email: parsed.data.email,
          password: parsed.data.password,
          mfa_code: parsed.data.mfa_code,
        });
        
        if (result.error) return null;
        
        return {
          id: result.data.user.id,
          email: result.data.user.email,
          name: result.data.user.display_name,
          tenant_id: result.data.user.tenant_id,
          roles: result.data.user.roles,
          permissions: result.data.user.permissions,
          access_token: result.data.access_token,
          refresh_token: result.data.refresh_token,
        };
      },
    }),
  ],
  callbacks: {
    async jwt({ token, user, account }) {
      if (user) {
        token.accessToken = user.access_token;
        token.refreshToken = user.refresh_token;
        token.tenantId = user.tenant_id;
        token.roles = user.roles;
        token.permissions = user.permissions;
      }
      
      // Refresh token if expired
      if (Date.now() > (token.accessTokenExpires as number)) {
        return await refreshAccessToken(token);
      }
      return token;
    },
    async session({ session, token }) {
      session.accessToken = token.accessToken;
      session.tenantId = token.tenantId;
      session.roles = token.roles;
      session.permissions = token.permissions;
      return session;
    },
  },
  pages: {
    signIn: '/login',
    error: '/login',
    newUser: '/onboarding',
  },
  session: {
    strategy: 'jwt',
    maxAge: 15 * 60, // 15 minutes
    updateAge: 5 * 60, // 5 minutes
  },
};
```

### 10.2 Permission-Based Rendering

```tsx
// src/components/providers/AuthProvider.tsx
'use client';

import { SessionProvider } from 'next-auth/react';
import { useAppStore } from '@/lib/stores/app-store';

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const { featureFlags, setFeatureFlags } = useAppStore();
  
  // Sync feature flags from session
  // ... implementation
  
  return (
    <SessionProvider>
      {children}
    </SessionProvider>
  );
}

// Permission hook
// src/lib/hooks/usePermissions.ts
export function usePermissions() {
  const { data: session } = useSession();
  const { isFeatureEnabled } = useAppStore();
  
  return {
    hasPermission: (permission: string) => 
      session?.permissions?.includes(permission) ?? false,
    hasRole: (role: string) => 
      session?.roles?.includes(role) ?? false,
    hasAnyPermission: (permissions: string[]) => 
      permissions.some((p) => session?.permissions?.includes(p) ?? false),
    hasAllPermissions: (permissions: string[]) => 
      permissions.every((p) => session?.permissions?.includes(p) ?? false),
    isFeatureEnabled,
  };
}

// Usage in components
function AssetCreateButton() {
  const { hasPermission } = usePermissions();
  
  if (!hasPermission('assets:write')) return null;
  
  return <Button>Create Asset</Button>;
}
```

---

## 11. Internationalization (i18n)

### 11.1 Configuration

```typescript
// src/lib/i18n/config.ts
export const locales = ['en', 'ar', 'fr', 'de', 'es'] as const;
export const defaultLocale = 'en' as const;
export type Locale = (typeof locales)[number];

export const localeConfig: Record<Locale, { label: string; rtl: boolean }> = {
  en: { label: 'English', rtl: false },
  ar: { label: 'العربية', rtl: true },
  fr: { label: 'Français', rtl: false },
  de: { label: 'Deutsch', rtl: false },
  es: { label: 'Español', rtl: false },
};
```

### 11.2 Translation Files

```
src/lib/i18n/
├── locales/
│   ├── en.json
│   ├── ar.json
│   ├── fr.json
│   ├── de.json
│   └── es.json
└── index.ts
```

```json
// src/lib/i18n/locales/en.json
{
  "common": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "edit": "Edit",
    "create": "Create",
    "search": "Search",
    "filter": "Filter",
    "loading": "Loading...",
    "error": "Error",
    "noData": "No data available"
  },
  "assets": {
    "title": "Assets",
    "create": "Create Asset",
    "import": "Import Assets",
    "detail": "Asset Details",
    "relationships": "Relationships",
    "history": "History",
    "maintenance": "Maintenance",
    "compliance": "Compliance",
    "fields": {
      "name": "Name",
      "type": "Type",
      "serialNumber": "Serial Number",
      "state": "State",
      "owner": "Owner",
      "location": "Location"
    }
  }
}
```

### 11.3 RTL Support (Arabic)

```css
/* src/styles/globals.css */
:root {
  /* Logical properties for RTL support */
  --spacing-inline-start: padding-inline-start;
  --spacing-inline-end: padding-inline-end;
}

[dir="rtl"] {
  /* Tailwind RTL plugin handles most, but custom: */
  .sidebar {
    transform: translateX(100%);
  }
  .sidebar.open {
    transform: translateX(0);
  }
}
```

---

## 12. Accessibility (a11y)

### 12.1 WCAG 2.1 AA Checklist

| Criterion | Implementation |
|-----------|----------------|
| **1.1.1 Non-text Content** | All images have `alt`; icons use `aria-hidden` with text labels |
| **1.3.1 Info and Relationships** | Semantic HTML (`<nav>`, `<main>`, `<aside>`, `<section>`, headings h1-h6) |
| **1.4.3 Contrast (Minimum)** | Tailwind color palette meets 4.5:1; dark mode tested |
| **1.4.4 Resize Text** | Rem units; browser zoom to 200% works |
| **2.1.1 Keyboard** | All interactive elements focusable; visible focus rings |
| **2.1.2 No Keyboard Trap** | Modal focus trap; `Tab`/`Shift+Tab` navigation |
| **2.4.3 Focus Order** | Logical tab order; `focus-visible` styles |
| **2.4.6 Headings and Labels** | Descriptive headings; form labels associated |
| **3.2.1 On Focus** | No context change on focus alone |
| **3.3.2 Labels or Instructions** | Form fields have labels; error messages linked via `aria-describedby` |
| **4.1.2 Name, Role, Value** | Custom components use Radix UI primitives with proper ARIA |

### 12.2 Accessible Component Patterns

```tsx
// Accessible Data Table
// src/components/tables/DataTable.tsx
'use client';

import { useReactTable, getCoreRowModel, flexRender } from '@tanstack/react-table';
import { createColumnHelper } from '@tanstack/react-table';

export function DataTable<T>({ 
  columns, 
  data, 
  ariaLabel,
  pagination = true 
}: DataTableProps<T>) {
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    // ... other features
  });
  
  return (
    <div className="overflow-x-auto" role="region" aria-label={ariaLabel} tabIndex={0}>
      <table className="w-full" role="grid">
        <thead>
          {table.getHeaderGroups().map((headerGroup) => (
            <tr key={headerGroup.id} role="row">
              {headerGroup.headers.map((header) => (
                <th
                  key={header.id}
                  role="columnheader"
                  scope="col"
                  aria-sort={header.getIsSorted() ? (header.getSortDirection() === 'asc' ? 'ascending' : 'descending') : 'none'}
                  className="..."
                >
                  {header.isPlaceholder ? null : flexRender(header.column.columnDef.header, header.getContext())}
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody>
          {table.getRowModel().rows.map((row) => (
            <tr key={row.id} role="row">
              {row.getVisibleCells().map((cell) => (
                <td key={cell.id} role="gridcell">
                  {flexRender(cell.column.columnDef.cell, cell.getContext())}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
      {pagination && <TablePagination table={table} />}
    </div>
  );
}
```

---

## 13. Performance Optimization

### 13.1 Performance Budgets

| Metric | Budget | Measurement |
|--------|--------|-------------|
| **LCP (Largest Contentful Paint)** | < 2.5s | Core Web Vitals |
| **CLS (Cumulative Layout Shift)** | < 0.1 | Core Web Vitals |
| **INP (Interaction to Next Paint)** | < 200ms | Core Web Vitals |
| **TBT (Total Blocking Time)** | < 200ms | Lighthouse |
| **Bundle Size (JS gzipped)** | < 200KB | Webpack Bundle Analyzer |
| **Initial HTML Size** | < 50KB | Network tab |
| **Time to Interactive** | < 3.5s | Lighthouse |
| **First Contentful Paint** | < 1.8s | Core Web Vitals |

### 13.2 Optimization Techniques

| Technique | Implementation |
|-----------|----------------|
| **Code Splitting** | Dynamic `import()` for heavy components; route-based splitting automatic |
| **Tree Shaking** | ES modules; sideEffects: false in package.json |
| **Image Optimization** | Next.js `<Image>` component; WebP/AVIF; responsive sizes |
| **Font Optimization** | Self-hosted fonts; `font-display: swap`; preload critical |
| **CSS Optimization** | Tailwind JIT; purge unused; critical CSS inlined |
| **Server Components** | Default; reduces client bundle by 60%+ |
| **Streaming** | `Suspense` boundaries for slow data; `loading.tsx` |
| **Prefetching** | `<Link prefetch>` for likely next pages |
| **Caching** | `next: { cache: 'force-cache' }` for static; `revalidate` for ISR |
| **Compression** | Brotli (Cloudflare); gzip fallback |

### 13.3 Bundle Analysis

```bash
# Analyze bundle
npm run build && npx @next/bundle-analyzer

# Key targets:
# - @nexusasset/ui: < 50KB
# - @nexusasset/sdk: < 30KB
# - Chart libraries: < 40KB (lazy loaded)
# - Framer Motion: < 20KB (lazy loaded)
```

---

## 14. Testing Strategy

### 14.1 Test Pyramid

```
                    ┌─────────────┐
                    │   E2E       │  ← 10% (Playwright)
                    │  (Critical  │     Happy paths, auth, billing
                    │   flows)    │
                   ┌───────────────┐
                   │  Integration  │  ← 20% (Vitest + MSW)
                   │  (Components  │     Component + API integration
                   │   + Hooks)    │
                  ┌─────────────────┐
                  │    Unit         │  ← 70% (Vitest)
                  │  (Utils, Hooks, │     Pure functions, validators,
                  │   Components)   │     schema parsing
                  └─────────────────┘
```

### 14.2 Test Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
    include: ['src/**/*.test.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 70,
        statements: 80,
      },
    },
    globals: true,
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### 14.3 Playwright E2E Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

---

## 15. Deployment & CI/CD

### 15.1 Dockerfile

```dockerfile
# frontend/Dockerfile
FROM node:22-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci

# Generate Prisma/OpenAPI types
COPY src/lib/sdk ./src/lib/sdk
RUN npm run generate:types

# Build
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED=1
RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]
```

### 15.2 CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/frontend.yml
name: Frontend CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}/frontend

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm run test:unit -- --coverage
      - run: npm run test:component
      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  build-and-test-e2e:
    needs: lint-and-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - run: npm run test:e2e
        env:
          PLAYWRIGHT_BASE_URL: http://localhost:3000

  docker-build:
    needs: build-and-test-e2e
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: docker-build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Staging
        run: |
          kubectl set image deployment/nxa-frontend \
            nxa-frontend=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n nxa-frontend

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Promote via ArgoCD
        run: |
          argocd app set nxa-frontend-prod --parameter image.tag=${{ github.sha }}
          argocd app sync nxa-frontend-prod
```

---

## 16. Security Considerations

| Concern | Mitigation |
|---------|------------|
| **XSS** | React auto-escapes; `dangerouslySetInnerHTML` banned; CSP header |
| **CSRF** | NextAuth.js CSRF protection; SameSite cookies |
| **Clickjacking** | `X-Frame-Options: DENY`; CSP `frame-ancestors 'none'` |
| **Mixed Content** | All assets HTTPS; HSTS header |
| **Sensitive Data** | No secrets in client bundle; runtime config from secure endpoint |
| **Auth Token Storage** | HttpOnly cookies for refresh token; JWT in memory only |
| **Dependency Scanning** | `npm audit` + Snyk in CI; automated PRs for updates |
| **Content Security Policy** | Strict CSP: `default-src 'self'; script-src 'self' 'unsafe-inline' https://cdn.nexusasset.cloud;` |

---

## 17. Diagrams

| Diagram ID | Title | Type | Location | Description |
|------------|-------|------|----------|-------------|
| **DIA-092** | Frontend Architecture | Component | Section 1.1 | Edge → Next.js → Backend/BFF |
| **DIA-093** | Rendering Strategy Matrix | Table | Section 4.1 | Page → Strategy → Reason |
| **DIA-094** | State Management Layers | Layered | Section 5.1 | Server/Client/Form/URL/Server Cache |
| **DIA-095** | BFF Integration Flow | Sequence | Section 6 | Frontend → BFF → Multiple Backend |
| **DIA-096** | Micro-Frontend Loading | Sequence | Section 7.2 | Manifest → Remote Entry → Component |
| **DIA-097** | Component Hierarchy | Tree | Section 8.1 | Component organization |
| **DIA-098** | Route Structure | Tree | Section 9.1 | App Router structure |
| **DIA-099** | Auth Flow | Sequence | Section 10.1 | Login → JWT → Session → Refresh |
| **DIA-100** | i18n/RTL Flow | Flowchart | Section 11.3 | Locale detection → Translation → RTL |
| **DIA-101** | Performance Budget | Component | Section 13.1 | Metrics → Targets → Tools |
| **DIA-102** | Test Pyramid | Pyramid | Section 14.1 | Unit → Integration → E2E |
| **DIA-103** | Deployment Pipeline | Flowchart | Section 15.2 | CI → Build → Staging → Production |

---

## 18. Tables

| Table ID | Title | Location | Row Count |
|----------|-------|----------|-----------|
| **TBL-191** | Design Principles | Section 1.2 | 8 |
| **TBL-192** | Tech Stack | Section 2 | 24 |
| **TBL-193** | Rendering Decision Matrix | Section 4.1 | 10 |
| **TBL-194** | State Management Layers | Section 5.1 | 5 |
| **TBL-195** | TanStack Query Config | Section 5.2 | 1 |
| **TBL-196** | BFF Endpoints | Section 6.1 | 3 |
| **TBL-197** | MFE Manifest | Section 7.3 | 1 |
| **TBL-198** | Component Design Standards | Section 8.2 | 8 |
| **TBL-199** | Route Structure | Section 9.1 | 1 |
| **TBL-200** | NextAuth Config | Section 10.1 | 1 |
| **TBL-201** | Locale Config | Section 11.1 | 5 |
| **TBL-202** | WCAG 2.1 AA Checklist | Section 12.1 | 11 |
| **TBL-203** | Performance Budgets | Section 13.1 | 8 |
| **TBL-204** | Optimization Techniques | Section 13.2 | 10 |
| **TBL-205** | Test Pyramid Distribution | Section 14.1 | 3 |
| **TBL-206** | Security Mitigations | Section 16 | 8 |
| **TBL-207** | Diagram Index | Section 17 | 12 |
| **TBL-208** | Table Index | Section 18 | 18 |

---

---

**End of DOC-09 — NexusAsset Cloud Frontend Architecture**

*Document Control: Version 1.0.0 | Classification: Internal — Confidential | Next Review: 2026-12-09*

*For questions or updates: docs@nexusasset.cloud | Frontend Team: frontend@nexusasset.cloud*