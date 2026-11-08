# Phase 05 — Frontend (Next.js + React + BFF)

## Objective
Implement the NexusAsset Cloud web application in `frontend/` using Next.js 15 (App Router), React 19, TypeScript, shadcn/ui, and Tailwind CSS, with a Backend For Frontend (BFF) layer aligned to OpenAPI contracts.

## Preconditions
- Phases 01–03 complete. Phase 04 API/integration surfaces are available or stubbed consistently.
- `api/contracts/openapi.yaml` is the source of truth.

## Required Context
- `frontend/`
- `packages/@nexusasset/*` (api-client, shared-types, design-tokens, ui, configs)
- `api/contracts/openapi.yaml`
- `10-design-system/` and `09-api/` docs

## Deliverables
1. **Application foundation**
   - Next.js 15 App Router + React 19 + TypeScript strict configuration.
   - Tailwind/shadcn/ui setup, design-token integration, global layout, theming, and RTL-ready structure (Arabic default/English supported as project standard).
2. **BFF/API integration**
   - Implement BFF route handlers/server actions as appropriate for authenticated API proxying, session/token handling, and UI-specific aggregation.
   - Use generated or typed API client patterns from `@nexusasset/api-client`/OpenAPI; do not hardcode ad hoc contract shapes where shared types exist.
3. **Core user flows (baseline)**
   - Authentication/session bootstrap UI.
   - Tenant/workspace context display.
   - Dashboard shell/navigation.
   - Assets/CMDB list/detail baseline views.
   - Lansweeper integration/sync status baseline views.
   - Audit/notifications placeholders or read views where contracts exist.
4. **Quality and accessibility**
   - Component conventions, loading/error/empty states, form validation, accessible labels/keyboard behavior, and responsive layout.
   - ESLint, TypeScript, and test setup (Vitest/RTL/Playwright baseline as feasible).
5. **Configuration**
   - Use `NXA_*`/public runtime env conventions as applicable; never expose secrets to the browser.

## Verification
┌─ Specification Block (converted from `powershell`; scope 2 — no fenced code blocks) ─┐
│ cd frontend
│ pnpm install  # or repo-root workspace install
│ pnpm lint
│ pnpm typecheck
│ pnpm test
│ pnpm build
└──────────────────────────────────────────────────────────────────────────────┘
If Playwright/E2E is heavy, provide smoke E2E baseline and document full run command.

## Exit Criteria
- Frontend builds successfully.
- BFF/auth/contract integration pattern is established.
- Core CMDB/Lansweeper baseline screens exist with robust states.
- RTL/accessibility foundations are present.
