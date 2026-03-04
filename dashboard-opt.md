## Dashboard Optimization Plan

### Overview

This document captures improvements for the IFRS S1/S2 dashboard, focusing on:
- Hardening **auth and access control** between client and server.
- Reducing **duplicate data fetching** and round-trips.
- Tightening **typing and contracts** between frontend and backend.
- Improving **performance** and **UX** for dashboard views.

The plan is structured into a short implementation phase with weekly/daily breakdowns.

---

## Current Integration – Quick Assessment

- **Client page**: `client/app/dashboard/page.tsx`
  - Uses `AssessmentDashboard`, `ReadinessScore`, `CategoryBreakdown`, `ProgressCharts`, `ComplianceMatrix`, `GapAnalysis`, and `ReportExport`.
  - Draws from `assessmentStore` (local answers/scores) and `authStore` (userId), with `autoFetch` flags to decide when to call backend APIs.
  - Uses skeletons and rich empty/error states; no obvious placeholders.

- **API client**: `client/lib/api/dashboardApi.ts`
  - Talks to:
    - `/api/dashboard/data` or `/api/dashboard/user/:userId/data`
    - `/api/dashboard/score` or `/user/:userId/score`
    - `/api/dashboard/progress` or `/user/:userId/progress`
    - `/api/dashboard/gaps` or `/user/:userId/gaps`
    - `/api/dashboard/compliance` or `/user/:userId/compliance`
  - Adds `Authorization: Bearer <accessToken>` when available and uses `credentials: 'include'`.

- **Server controller**: `server/src/controllers/dashboardController.ts`
  - `getDashboardData` aggregates:
    - Latest or specified assessment (answers + scores).
    - Readiness score (via `ScoringService`).
    - Progress (via `ProgressService`).
    - Compliance matrix (via `ComplianceMatrixService`).
    - Gap analysis (via `GapIdentificationService`).
    - Recent activity + historical trends via Prisma queries.
  - Lightweight endpoints (`getReadinessScore`, `getProgress`, `getGapAnalysis`, `getComplianceMatrix`) reuse underlying services.
  - Returns structured empty dashboard data when no assessment exists (not a placeholder).

- **Routes**: `server/src/routes/dashboard.ts`
  - Exposes `/api/dashboard/*` and `/api/dashboard/user/:userId/*` without explicit auth middleware (relies on upstream `req.user` if present).

Overall, the dashboard is **backed by real endpoints and services**; optimization is mainly about security, duplication, and consistency rather than filling gaps.

---

## Phase 1 – Security & Contracts (Week 1)

**Goal**: Ensure dashboard data is always scoped to the authenticated user, and client/server contracts are strongly typed.

### Week 1 – Daily Breakdown

- **Day 1 – Lock down dashboard routes**
  - Add `requireAuth` middleware to:
    - `GET /api/dashboard/data`
    - `GET /api/dashboard/score`
    - `GET /api/dashboard/progress`
    - `GET /api/dashboard/gaps`
    - `GET /api/dashboard/compliance`
  - Decide policy for `/user/:userId/*` routes:
    - Option A: Remove them and always rely on `req.user.id`.
    - Option B: Keep only for admin use:
      - Add `requireAuth` + role/permission check (e.g. `isAdmin` or `canViewUserDashboards`).
      - Validate that `userId` matches `req.user.id` unless admin.

- **Day 2 – Harden controller logic**
  - In `DashboardController`, ensure `userId` is derived as:
    - Normal user: `const userId = (req as any).user?.id`.
    - Only allow `req.params.userId` when the caller is an authorized admin.
  - Standardize error responses:
    - `401` with `{ error: 'Authentication required', code: 'DASHBOARD_UNAUTHORIZED' }`.
    - `403` with `{ error: 'Forbidden', code: 'DASHBOARD_FORBIDDEN' }` where applicable.

- **Day 3 – Tighten client types (GapAnalysis + ComplianceMatrix)**
  - Replace `any` fields in `GapAnalysis` (e.g. `highGaps`, `mediumGaps`, `lowGaps`, `byCategory`) with a shared `GapItem` type that matches server output.
  - Ensure `ComplianceMatrix` types in `dashboardApi.ts` align precisely with `ComplianceMatrixService` (categories, levels, mandatory flags).
  - Update `GapAnalysis` and `ComplianceMatrix` components to use the stricter types; fix any mismatches discovered.

- **Day 4 – Error code alignment**
  - Update dashboard endpoints to include a `code` field on JSON errors (e.g. `DASHBOARD_NO_ASSESSMENT`, `DASHBOARD_ERROR`).
  - Extend `parseError` to recognize dashboard-specific error codes and map them to clear user messages and `ErrorInfo.type`.

- **Day 5 – Quick security regression tests**
  - Verify that:
    - Unauthenticated requests to `/api/dashboard/*` return `401`.
    - Non-admin users cannot access other users’ dashboards, even via `/user/:userId/*`.
    - Existing client dashboard flows still work for a logged-in user.

### Week 1 – Implementation Summary (Done)

- **Routes** (`server/src/routes/dashboard.ts`): `requireAuth` added to all dashboard routes (`/data`, `/score`, `/progress`, `/gaps`, `/compliance` and `/user/:userId/*` variants). Unauthenticated requests are rejected before reaching the controller.
- **Controller** (`server/src/controllers/dashboardController.ts`): Introduced `getEffectiveUserId(req)` that uses `req.user.userId` only (set by `requireAuth`). For `/user/:userId/*` routes, returns 403 if `params.userId !== req.user.userId`. All error responses include a `code`: `DASHBOARD_UNAUTHORIZED` (401), `DASHBOARD_FORBIDDEN` (403), `DASHBOARD_ERROR` (500).
- **Client types** (`client/lib/api/dashboardApi.ts`): Added `GapItem` interface and replaced all `Array<any>` in `GapAnalysis` with `GapItem[]`. `ComplianceMatrix` already matched server (level, mandatory). Added `throwDashboardError()` so non-OK responses parse JSON and throw an error with a `code` property for the client error handler.
- **Error handler** (`client/lib/utils/errorHandler.ts`): Extended `parseError()` to recognize `DASHBOARD_UNAUTHORIZED`, `DASHBOARD_FORBIDDEN`, and `DASHBOARD_ERROR` via `getErrorCode(error)` and return appropriate `ErrorInfo` (type `auth` / `forbidden` / `server` with clear user messages). Added `forbidden` to `ErrorInfo.type`.
- **Verification**: Run client and server builds; unauthenticated `GET /api/dashboard/score` returns 401; with valid token, dashboard data is returned. No push performed.

---

## Phase 2 – Fetch Strategy & Performance (Week 2)

**Goal**: Reduce duplicate requests and improve perceived dashboard performance by centralizing data fetching where appropriate.

### Week 2 – Daily Breakdown

- **Day 1 – Inventory current dashboard requests**
  - Instrument or log client calls from `useDashboardApi` and `DashboardAPI` to see:
    - Which routes are called on initial dashboard load.
    - Whether `score`, `progress`, `gaps`, and `compliance` are all fetched in parallel for the same user/assessment.
  - Document the current request sequence for the main dashboard page.

- **Day 2 – Introduce `useDashboardData` hook**
  - Add a new React Query hook (e.g. `useDashboardData`) that calls `DashboardAPI.getDashboardData()` once and returns:
    - `readinessScore`
    - `progress`
    - `complianceMatrix`
    - `gapAnalysis`
    - `historicalTrends`, `recentActivity`
  - Optionally, keep separate `useReadinessScore`/`useProgressHistory` hooks for lightweight, stand-alone widgets used outside the dashboard page.

- **Day 3 – Wire dashboard components to shared data**
  - On the main `dashboard/page.tsx`, use `useDashboardData` when:
    - The user is logged in.
    - No fresh local `assessmentStore` data is available.
  - Pass slices of the returned data as props to:
    - `ReadinessScore`
    - `ProgressCharts`
    - `ComplianceMatrix` (with `useDashboardApi=false` in this case)
    - `GapAnalysis`
  - Keep existing props-based behavior when local `assessmentStore` data is present (to avoid extra network hops just after an assessment).

- **Day 4 – Server-side micro-optimizations**
  - In `DashboardController.getDashboardData`:
    - Ensure queries for `recentActivity` and `historicalTrends` use only required fields.
    - Consider using `Promise.all` to parallelize non-dependent Prisma calls (recent assessments, historical assessments) after you have the main `assessment`.
  - Optionally, add a short-lived Redis cache for `DashboardData` keyed by `(userId, assessmentId)` with a TTL of 60–120 seconds for frequently refreshed dashboards.

- **Day 5 – Performance validation**
  - Measure:
    - Number of HTTP requests on initial dashboard load (before vs. after).
    - Time-to-render for readiness score, progress chart, compliance matrix, and gap analysis.
  - Validate that skeletons still show sensibly, and that the dashboard remains responsive even on slow connections.

### Week 2 – Implementation Summary (Done)

- **Dashboard page** (`client/app/dashboard/page.tsx`): Uses `useDashboardData(userId, assessmentId)` when `!!userId` so a single `GET /api/dashboard/data` (or user-scoped variant) runs. Derives `effectiveScores`, `hasServerData`, and `progressHistory` from the shared payload. Passes `effectiveScores` to ReadinessScore and CategoryBreakdown with `autoFetch={!effectiveScores}`; passes `progressHistory` to ProgressCharts with `autoFetch={!hasServerData}`; passes `dashboardData?.complianceMatrix` and `dashboardData?.gapAnalysis` where supported. Empty state shows only when `!hasData && !hasServerData`. Gap Analysis section renders when `hasData || hasServerData` so server-only users see it.
- **ComplianceMatrix** (`client/components/assessment/ComplianceMatrix.tsx`): Added optional `complianceMatrixFromDashboard` prop. When set, the component uses it (converted via `dashboardMatrixToCompliance`) and skips both ComplianceAPI and DashboardAPI calls. Page passes `dashboardData?.complianceMatrix` and sets `useDashboardApi` only when that prop is not present.
- **Server controller** (`server/src/controllers/dashboardController.ts`): In `getDashboardData`, score, progress, compliance matrix, and gap analysis are computed in parallel with `Promise.all`. Recent activity and historical trends are fetched in parallel with a second `Promise.all`. Historical trends query uses `select` with only required fields and `scores: { select: { score: true } }` to limit data.

---

## Phase 3 – UX & Navigation Polishing (Week 3)

**Goal**: Improve the user experience around navigation, empty states, and failure modes.

### Week 3 – Daily Breakdown

- **Day 1 – Navigation upgrades**
  - Replace `window.location.href = '/'` in the dashboard header with Next.js navigation (`useRouter().push('/')`) for smoother transitions.
  - Ensure the “Start Assessment” CTA in the dashboard empty state either:
    - Navigates to the in-chat assessment entry point (e.g. `/` with a query or state flag), or
    - Uses a dedicated route that clearly starts an assessment.

- **Day 2 – Page-level error handling**
  - Add a light dashboard-specific error boundary or top-level error state:
    - If all key widgets (score, progress, gaps, compliance) fail to load due to server errors, show a single “Dashboard currently unavailable” banner at the top.
    - Provide a retry action that re-triggers the underlying queries or `useDashboardData`.

- **Day 3 – Empty and partial-data states**
  - Review combinations:
    - No assessments at all.
    - Only partial scores (e.g. incomplete assessment).
    - Completed assessment but missing historical trends.
  - Ensure each widget:
    - Shows an appropriate “no data yet” state (not a generic error).
    - Does not block rendering of other widgets that do have data.

- **Day 4 – Mobile/Responsive behavior**
  - QA the dashboard on mobile and tablet:
    - Check scroll behavior in the dashboard layout (charts, grids).
    - Ensure charts and grids remain usable and legible on narrow viewports.
  - Tweak `grid` breakpoints or chart sizing if any layout issues appear.

- **Day 5 – Documentation & follow-ups**
  - Update `dashboard-opt.md` and/or `OPTIMIZATIONS.md` with:
    - Final request patterns.
    - Auth rules for dashboard endpoints.
    - Any remaining ideas (e.g. admin-only extended views, PDF export performance).
  - Create follow-up tasks for any remaining stretch items (e.g. advanced filtering, per-assessment dashboard view).

