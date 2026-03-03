# Optimizations Report – Database, History, Error Handling & Logging

## Overview

This document captures recommended optimizations for the Complyx platform, covering database, persistence, security, error handling, logging, scroll smoothness, and loading UX for both web and mobile views.

---

## 1. Chat History Persistence

| Aspect | Details |
|--------|---------|
| **Overview** | Chat messages are persisted server-side for logged-in users; anonymous users keep local-only behavior. |
| **Implemented** | `ChatSession` and `ChatMessage` models (Prisma); `chatHistoryService` (create/get session, save message, get messages, list sessions); API under `/api/chat/sessions` and `/api/chat/sessions/:id/messages`; client `chatHistoryApi`, `useChatHistorySync` hooks, and page integration for load/persist. See Phase 5 Week 8 summary below. |
| **Impact** | History survives device changes and browser data loss; cross-device access; foundation for search, audit, and analytics. |

---

## 2. Redis Utilization

| Aspect | Details |
|--------|---------|
| **Overview** | Redis is used for token blacklist (logout revocation) and read-through cache for questions/categories. |
| **Implemented** | Shared client (`redisClient.ts`), token blacklist (`tokenBlacklistService.ts`), auth middleware checks blacklist; `redisCache.ts` used for `GET /api/questions` and `GET /api/questions/categories` (60s TTL). See Phase 4 Week 7 summary below. |
| **Impact** | Faster responses, lower DB load, proper logout, foundation for rate limiting and queues. |

---

## 3. Database Indexes

| Aspect | Details |
|--------|---------|
| **Overview** | Indexes exist mainly on `AuditLog`. Common query paths on `Assessment`, `Answer`, and `Question` are not indexed. |
| **Work to be done** | Add indexes on `Assessment(userId)`, `Assessment(createdAt)`, `Assessment(status)`, `Answer(assessmentId)`, `Answer(questionId)`, `Question(categoryId)`; create migrations. |
| **Impact** | Faster lookups, better scalability, lower DB load. |

---

## 4. Token Storage Security

| Aspect | Details |
|--------|---------|
| **Overview** | Access and refresh tokens are stored in `localStorage`, which is exposed to XSS. |
| **Work to be done** | Move refresh token to HttpOnly, Secure, SameSite cookie; keep access token in memory or short-lived cookie; update auth API and CORS/cookie settings. |
| **Impact** | Reduced token theft risk and stronger security posture. |

---

## 5. Session/Token Revocation

| Aspect | Details |
|--------|---------|
| **Overview** | Logout blacklists the access token in Redis; auth middleware rejects revoked tokens. |
| **Implemented** | `tokenBlacklistService.revokeToken` on logout; `requireAuth` middleware calls `isTokenRevoked` before accepting token. TTL matches access token expiry. |
| **Impact** | Logout actually invalidates tokens; ability to revoke compromised tokens. |

---

## 6. Data Retention Policy

| Aspect | Details |
|--------|---------|
| **Overview** | A scheduled retention job runs every 24h and deletes old audit logs, chat sessions/messages, and assessments per env-configured retention days. |
| **Implemented** | `server/src/jobs/scheduler.ts`, `retentionCleanupJob.ts`, `registerJobs.ts`; defaults 18/12/24 months (audit/chat/assessment); `RETENTION_*_DAYS` and `RETENTION_CLEANUP_ENABLED` env vars. See Phase 5 Week 9 summary below. |
| **Impact** | Controlled storage growth, better compliance, clearer data lifecycle. |

---

## 7. Error Handling

| Aspect | Details |
|--------|---------|
| **Overview** | Root `ErrorBoundary` exists, but API errors, async failures, and user-facing feedback are inconsistent. No global Next.js error handling. |
| **Work to be done** | • Add `app/error.tsx` and `app/global-error.tsx` for Next.js error UI<br>• Centralize API error handling (e.g. wrapper or interceptor) with user-friendly messages<br>• Add toast/notification system for inline errors (auth, chat, assessment)<br>• Standardize error codes and messages (client + server)<br>• Add retry logic for transient failures (network, 5xx)<br>• Improve ErrorBoundary: report to backend, optional recovery actions |
| **Impact** | Clearer feedback, fewer silent failures, easier debugging, better UX. |

---

## 8. Logging – Web View

| Aspect | Details |
|--------|---------|
| **Overview** | Client uses `console.error` in ErrorBoundary; no structured logging or remote reporting for web. |
| **Work to be done** | • Add client-side logger (e.g. `lib/logger.ts`) with levels (debug, info, warn, error)<br>• In dev: log to console; in prod: send to backend or external service (e.g. Sentry, Logtail)<br>• Add correlation IDs (e.g. `requestId`) for API calls<br>• Log key events: auth, chat send/receive, assessment start/submit, navigation errors<br>• Sanitize logs (no tokens, passwords, PII)<br>• Add `user-agent` and viewport info for web context |
| **Impact** | Better debugging, faster incident response, clearer production diagnostics. |

---

## 9. Logging – Mobile View

| Aspect | Details |
|--------|---------|
| **Overview** | App is responsive; mobile view uses same client code. Extend existing logger with lightweight mobile context—no separate mobile logging infrastructure. |
| **Work to be done** | • Extend logger with optional context: `isMobile`, `viewport`, `connection` (navigator.connection when available)<br>• Use `useIsMobile` or media query to set `isMobile`<br>• Batch or sample non-error logs in production to limit volume<br>• Sanitize logs (no tokens, passwords, PII) |
| **Impact** | Easier mobile debugging without added complexity. |

---

## 10. Scroll Smoothness

| Aspect | Details |
|--------|---------|
| **Overview** | Chat uses `CustomScrollbar` with `scrollBehavior: smooth` and `-webkit-overflow-scrolling: touch`. Small tweaks improve feel on mobile. |
| **Work to be done** | • Add `overscroll-behavior: contain` to scroll container<br>• Use passive scroll listeners where applicable<br>• Debounce `scrollToBottom` when many messages arrive quickly<br>• Reserve `will-change` for animated elements only, not scroll container |
| **Impact** | Smoother scrolling on mobile, fewer janks. |

---

## 11. Loading UX

| Aspect | Details |
|--------|---------|
| **Overview** | `LoadingScreen` exists; inline loading for chat and assessment can be improved with skeletons and optimistic updates. |
| **Work to be done** | • Add message skeleton (2–3 bubbles) while chat loads<br>• Add question skeleton for assessment question fetch<br>• Enforce minimum display time (e.g. 300ms) for loading states<br>• Add `loading.tsx` for route transitions<br>• Optimistic UI for chat send (show message immediately) |
| **Impact** | Faster perceived load, clearer feedback. |

---

## Summary Table

| # | Recommendation | Effort | Impact | Priority |
|---|----------------|--------|--------|----------|
| 1 | Chat history persistence | High | High | P1 |
| 2 | Redis utilization | Medium | High | P1 |
| 3 | Database indexes | Low | Medium | P0 |
| 4 | Token storage security | Medium | High | P1 |
| 5 | Session/token revocation | Medium | High | P1 |
| 6 | Data retention policy | Medium | Medium | P2 |
| 7 | Error handling | Medium | High | P1 |
| 8 | Logging – web view | Medium | Medium | P1 |
| 9 | Logging – mobile view | Low | Medium | P2 |
| 10 | Scroll smoothness | Low | Medium | P2 |
| 11 | Loading UX | Medium | Medium | P1 |

---

## Suggested Implementation Order

1. **Phase 1 (quick wins)**  
   Database indexes (3), scroll smoothness (10).

### Phase 1 – Weekly & Daily Implementation Plan

#### Week 1 – Database Indexes (3)

- **Goal**: Add and validate high‑impact indexes for `Assessment`, `Answer`, and `Question` without harming write performance.

- **Day 1 – Analyze query patterns**
  - Review Prisma services and controllers that hit `Assessment`, `Answer`, and `Question`.
  - Identify top read paths (e.g. get assessments for user, load answers for assessment, list questions by category).
  - Draft target indexes (columns, expected selectivity) and note any composite index candidates.

- **Day 2 – Design indexes & migration**
  - Update `schema.prisma` with `@@index` / `@@unique` for:
    - `Assessment(userId)`, `Assessment(createdAt)`, `Assessment(status)`
    - `Answer(assessmentId)`, `Answer(questionId)`
    - `Question(categoryId)`
  - Generate a Prisma migration and review the generated SQL.

- **Day 3 – Apply to dev database**
  - Run the migration against the local/dev DB.
  - Verify Prisma Client builds and basic CRUD still work.
  - Check index presence via `EXPLAIN`/`EXPLAIN ANALYZE` on key queries.

- **Day 4 – Benchmark & adjust**
  - Capture before/after timings for representative queries (API endpoints or direct SQL).
  - Confirm improved read performance and acceptable write overhead.
  - Adjust or drop any low‑value indexes identified by the benchmarks.

- **Day 5 – Cleanup & documentation**
  - Remove any obsolete indexes (including legacy ones on these tables, if redundant).
  - Document:
    - New indexes and rationale (Assessment: userId, createdAt, status; Answer: assessmentId, questionId; Question: categoryId)
    - Impact on read/write performance
    - How to extend index coverage for new features

#### Week 2 – Scroll Smoothness (10)

- **Goal**: Improve perceived scroll performance in chat and other scrollable views, especially on mobile.

- **Day 1 – Audit scroll containers**
  - Identify all components using custom scroll behavior (e.g. `CustomScrollbar`, chat message list, side panels).
  - Note current CSS props (`overflow`, `scroll-behavior`, `-webkit-overflow-scrolling`) and JS helpers (`scrollToBottom`).

- **Day 2 – CSS tweaks**
  - Add `overscroll-behavior: contain` to the main chat scroll container and other key scrollable areas where appropriate.
  - Ensure `-webkit-overflow-scrolling: touch` is applied only where it helps on iOS.
  - Verify no layout shift or regression on desktop.

- **Day 3 – Scroll handler optimization**
  - Convert any scroll listeners to passive where safe (`{ passive: true }`).
  - Debounce or throttle `scrollToBottom` when multiple messages arrive rapidly.
  - Ensure scroll helpers bail out early when the user is intentionally scrolled up (don’t auto‑snap down).

- **Day 4 – Will‑change & animation review**
  - Audit use of `will-change`; restrict it to elements that are actually animated.
  - Remove `will-change` from the main scroll container to avoid unnecessary GPU promotion.
  - Validate that chat animations (e.g. new message fade/slide) still look correct.

- **Day 5 – Cross‑device QA & docs**
  - Test scroll behavior on:
    - Desktop (Chrome, Safari)
    - Mobile Safari (iOS), Chrome on Android (if available)
  - Capture any remaining jank cases and either fix or log follow‑ups.
  - Document:
    - Scroll CSS choices (CustomScrollbar: smooth behavior, touch scrolling, overscroll containment)
    - Rules for when auto-scroll is enabled (only when user is near bottom; pause when scrolled up)
    - Best practices for adding new scrollable areas (reuse CustomScrollbar and prefer passive scroll listeners)

2. **Phase 2 (observability)**  
   Error handling (7), logging – web (8), logging – mobile (9).

### Phase 2 – Weekly & Daily Implementation Plan

#### Week 3 – Error Handling (7)

- **Goal**: Provide consistent, user-friendly error handling across pages and API calls.

- **Day 1 – Global error surfaces (Next.js)**
  - Add `app/error.tsx` for route-level errors (per Next.js app directory).
  - Add `app/global-error.tsx` to handle uncaught runtime errors with a branded fallback.
  - Ensure both surfaces show helpful copy and a “Try again” / “Back to chat” action.

- **Day 2 – API error wrapper on the client**
  - Create a small helper (e.g. `lib/api/client.ts`) that wraps `fetch`:
    - Normalizes error shapes (`{ message, code, status }`).
    - Maps network/5xx errors to friendly messages.
  - Update 1–2 key API modules (chat, assessment) to use the wrapper as a pattern.

- **Day 3 – Toast/notification system**
  - Add or standardize a toast/notification component (e.g. under `components/ui`).
  - Wire auth, chat, and assessment flows to show toasts for:
    - Validation errors
    - Network/server errors
    - Non-critical warnings

- **Day 4 – Error codes & retry logic**
  - Define a small set of canonical error codes (e.g. `AUTH_INVALID`, `ASSESSMENT_CONFLICT`, `RAG_TIMEOUT`).
  - Update server endpoints (starting with chat/assessment) to return these codes.
  - Add simple retry logic in the API wrapper for clearly transient errors (e.g. network failures, 502/503) with a cap on attempts.

- **Day 5 – ErrorBoundary refinement & docs**
  - Ensure the existing React ErrorBoundary:
    - Logs via the shared client logger (with basic context like `source: 'ErrorBoundary'`).
    - Offers a clear recovery action (reload, go home, report issue).
  - Document:
    - How to throw typed errors from server/client.
    - How to surface them via global error components, toasts, and ErrorBoundary.

#### Week 4 – Logging – Web View (8) & Mobile View (9)

- **Goal**: Introduce structured, privacy-safe client logging for web and mobile views using a shared logger.

- **Day 1 – Logger foundation**
  - Create `lib/logger.ts` with log levels (`debug`, `info`, `warn`, `error`).
  - Implement environment-aware behavior:
    - In `development`: log to console with clear prefixes.
    - In `production`: send logs to a backend endpoint (stub) or external provider (behind a single function).

- **Day 2 – Web logging integration**
  - Replace scattered `console.error` calls in key areas (ErrorBoundary, chat page, auth flows) with the logger.
  - Ensure logs include:
    - A lightweight `context` object (e.g. `{ feature: 'chat', event: 'send' }`).
    - Optional `requestId`/correlation ID when available from the server.

- **Day 3 – Mobile-context extensions**
  - Extend logger to accept optional mobile-related context:
    - `isMobile`, `viewport` size, and `connection` info (from `navigator.connection` where available).
  - Use `useIsMobile` or an equivalent hook to set `isMobile` in log calls triggered from mobile layouts.
  - Add basic sampling/batching for non-error logs in production to avoid volume spikes.

- **Day 4 – Event coverage**
  - Define a short list of key client events to log:
    - Auth: login success/failure, logout.
    - Chat: message send, error, regenerate.
    - Assessment: start, submit, completion, error.
    - Navigation: route change errors.
  - Add logger calls in those code paths with consistent event names and contexts.

- **Day 5 – Privacy review & documentation**
  - Verify logs do **not** contain:
    - Tokens, passwords, or other secrets.
    - Full message content when not necessary (log counts/metadata instead where appropriate).
  - Update `OPTIMIZATIONS.md` or a dedicated logging doc with:
    - When to log (and at which level).
    - How to add new log events.
    - How mobile context is attached without special infrastructure.
    - Examples of safe contexts (e.g. `sessionId`, `assessmentId`, `status`, `isMobile`) vs. unsafe (raw chat text, passwords).

3. **Phase 3 (UX)**  
   Loading UX (11).

### Phase 3 – Weekly & Daily Implementation Plan

#### Week 5 – Loading UX (11)

- **Goal**: Improve perceived performance with skeletons, consistent loading states, and optimistic UI.

- **Day 1 – Inventory loading states**
  - Identify all major loading experiences:
    - Initial app load (`LoadingScreen`).
    - Chat history / first messages.
    - Assessment question fetch and completion summary.
    - Dashboard and admin views.
  - Decide where skeletons vs. simple spinners make the most sense.

- **Day 2 – Chat message skeletons**
  - Add a small set of skeleton components for chat (2–3 message bubbles with varying widths).
  - Show skeletons:
    - When opening a session that’s loading persisted chat history.
    - When reconnecting after a hard refresh, before messages rehydrate.

- **Day 3 – Assessment skeletons & route loading**
  - Add a question skeleton component for in-chat assessment (placeholder card with faux options).
  - Use route-level `loading.tsx` where helpful (e.g. dashboard) for smoother transitions.

- **Day 4 – Minimum loading time & optimistic send**
  - Ensure key loading states display for at least ~300 ms so users can see them and avoid “flash”.
  - Confirm chat send remains optimistic:
    - Message appears immediately.
    - Error state replaces or annotates the message if the request fails.

- **Day 5 – UX review & polish**
  - Test on slow 3G/4G throttling in devtools.
  - Adjust timings, opacity, and motion so loading feels smooth but not heavy.
  - Document:
    - Where skeletons are used.
    - When to introduce new skeletons for future features.

**Week 5 implementation notes (done):**
- **Chat skeleton** (`ChatSkeleton`): Shown when the chat is “loading” before the welcome message (session exists, no messages yet, welcome not sent). Keeps empty state from flashing.
- **Assessment question skeleton** (`AssessmentQuestionSkeleton`): Shown when in-chat assessment is in progress but the next question is not yet available (`in_progress`, no `currentQuestion`, `totalQuestions` > 0).
- **Completion summary**: `CompletionSummarySkeleton` is used when assessment is completed and the summary is still being fetched.
- **Route loading**: Root `app/loading.tsx` and `app/dashboard/loading.tsx` use `LoadingScreen` for route-level transitions.
- **Minimum display time**: Initial app load uses `usePageLoading` with a minimum display time (e.g. 1s). For other loading states (e.g. completion summary), aim for at least ~300 ms before switching to content to avoid flash.
- **Optimistic chat**: Chat send remains optimistic (user message is added immediately; errors are shown in place).

4. **Phase 4 (security)**  
   Token storage (4), session revocation (5), Redis (2).

### Phase 4 – Weekly & Daily Implementation Plan

#### Week 6 – Token Storage Security (4)

- **Goal**: Move toward HttpOnly refresh tokens and safer access-token handling.

- **Day 1 – Design & env review**
  - Decide final model:
    - Refresh token in HttpOnly, Secure, SameSite cookie.
    - Short-lived access token in memory (or non-HttpOnly cookie if needed).
  - Review current auth routes and CORS/cookie settings.

- **Day 2 – Server changes (refresh token)**
  - Update auth controller(s) to:
    - Set refresh token in an HttpOnly cookie on login.
    - Clear the cookie on logout.
  - Ensure cookie attributes are environment-aware (Secure only in HTTPS, proper SameSite).

- **Day 3 – Client changes (token handling)**
  - Stop storing refresh tokens in `localStorage`.
  - Keep access token in memory or short-lived storage as per design.
  - Update auth flows to work with cookie-based refresh (no direct access to refresh token on client).

- **Day 4 – Edge cases & CSRF considerations**
  - Review CSRF exposure with cookie-based tokens.
  - If needed, plan CSRF protections (e.g. same-site cookies + CSRF token for state-changing POSTs) for a later phase.

- **Day 5 – Testing & documentation**
  - Test login / refresh / logout across:
    - Browser refreshes.
    - Multiple tabs.
  - Update security section in docs with the new token storage approach and trade-offs.

#### Week 7 – Session/Token Revocation (5) & Redis Utilization (2)

- **Goal**: Make logout and token revocation effective, and start using Redis for auth/session-related concerns.

- **Day 1 – Redis client & config**
  - Add or confirm a shared Redis client module in the server.
  - Verify connectivity in local/dev (reusing the existing Docker Redis).

- **Day 2 – Token blacklist model**
  - Design a blacklist structure (e.g. key per token ID or jti with TTL matching token expiry).
  - Implement helper functions:
    - `revokeToken(tokenId, ttl)`
    - `isTokenRevoked(tokenId)`

- **Day 3 – Integrate blacklist with auth**
  - On logout, add current access (and possibly refresh) token IDs to the blacklist.
  - Update auth middleware to check blacklist and reject revoked tokens.

- **Day 4 – Additional Redis utilization**
  - Identify 1–2 read-heavy queries (e.g. questions, categories, dashboard summaries).
  - Add simple caching with TTL and cache invalidation rules.

- **Day 5 – Testing & monitoring**
  - Test:
    - Logout invalidates tokens immediately.
    - Revoked tokens cannot be reused.
  - Measure basic latency improvements for cached endpoints.
  - Document:
    - Redis usage patterns.
    - How to extend the blacklist and caching to future features.

**Week 7 – Implementation summary (done)**

- **Redis client**: `server/src/utils/redisClient.ts` – singleton client, `getRedisClient()`, connects to `REDIS_URL` (default `redis://localhost:6379`).
- **Token blacklist**: `server/src/services/auth/tokenBlacklistService.ts`
  - Keys: `token:blacklist:<sha256(token)>`, TTL = access token TTL (15 min).
  - `revokeToken(token, ttlSeconds)` – called on logout.
  - `isTokenRevoked(token)` – checked in auth middleware; fails open if Redis is down.
- **Auth middleware**: `server/src/middleware/authMiddleware.ts` – `requireAuth` extracts Bearer token, checks blacklist, verifies JWT, sets `req.user`. Used on `GET /api/auth/me`.
- **Logout**: Access token is blacklisted on logout; refresh token cookie is cleared. Cookie-parser added so refresh endpoint can read `refreshToken` from cookie.
- **Caching**: `server/src/utils/redisCache.ts` – `cacheGet`/`cacheSet`/`cacheKey` with 60s default TTL.
  - **Questions list**: `GET /api/questions` (no query filters) cached under `cache:questions:list`, 60s.
  - **Categories**: `GET /api/questions/categories` cached under `cache:questions:categories`, 60s.
- **Extending**: Add more cache keys via `cacheKey(['scope', 'id'])`; add blacklist check to any route that verifies access tokens by using `requireAuth` or calling `isTokenRevoked` before `verifyToken`.

5. **Phase 5 (features & operations)**  
   Chat history persistence (1), data retention (6).

### Phase 5 – Weekly & Daily Implementation Plan

#### Week 8 – Chat History Persistence (1)

- **Goal**: Persist chat messages server-side for logged-in users, enabling cross-device history and analytics.

- **Day 1 – Schema & model design**
  - Design `ChatSession` and `ChatMessage` models (or equivalents).
  - Decide retention defaults, relation to `User` and `Session`.

- **Day 2 – Migrations & repository**
  - Add models to Prisma schema and run migrations.
  - Implement a repository/service layer for chat persistence (save, list by session/user, pagination).

- **Day 3 – API endpoints**
  - Add endpoints for:
    - Saving messages.
    - Fetching a session’s messages.
    - Listing sessions for a user.

- **Day 4 – Client integration**
  - Update chat store to:
    - Load history on session selection.
    - Persist new messages for logged-in users.
  - Keep local-only behavior for anonymous users.

- **Day 5 – UX & performance**
  - Ensure loading of history uses skeletons and doesn’t block input.
  - Test with long histories and multiple sessions.

**Week 8 – Implementation summary (done)**

- **Schema**: `ChatSession` (userId, title, lastMessageAt) and `ChatMessage` (sessionId, userId, role, content, metadata); migration `add_chat_history_models`.
- **Server**: `server/src/services/chat/chatHistoryService.ts` – createSession, getSession, createOrGetSession, saveMessage, getMessages (cursor pagination), listSessions. `server/src/controllers/chatHistoryController.ts` – listSessions, createOrGetSession, getMessages, saveMessage (all require auth). Routes on `router.get/post('/sessions', ...)` and `router.get/post('/sessions/:sessionId/messages', ...)` under `/api/chat`, protected by `requireAuth`.
- **Client**: `client/lib/api/chatHistoryApi.ts` – listChatSessions, createOrGetChatSession, getChatMessages, saveChatMessage (Bearer token). `client/hooks/useChatHistorySync.ts` – useEnsureServerSession, useLoadSessionHistory, useFetchServerSessions, isServerSessionId. Session store: addServerSession, setSessionsFromServer. Chat store: setSessionMessagesFromServer. Home page: when logged in and no session, ensureServerSession(); when switching to a server session, loadSessionHistory(); when sending messages with a server session, saveChatMessage (user + assistant); welcome message persisted when logged in.
- **UX**: History load uses existing chat UI; messages fetched on session switch (limit 200). Anonymous users unchanged. Further: list server sessions in sidebar and skeleton when loading history can be added later.

#### Week 9 – Data Retention Policy (6)

- **Goal**: Define and enforce data lifecycle rules for audit logs, chat history, and old assessments.

- **Day 1 – Policy definition**
  - **AuditLog**: Retain **18 months** by default. This is long enough for most internal investigations and trend analysis, but bounded for storage and privacy.
  - **ChatMessage / ChatSession**: Retain **12 months** of chat history per user by default. Sessions with no activity for more than 12 months become eligible for cleanup.
  - **Assessment / Score / Answer**: Retain **24 months** by default to support year-on-year comparisons and regulatory lookback. Older completed assessments can be archived or hard-deleted depending on org requirements.
  - All default windows are **configurable via environment variables** (e.g. `RETENTION_AUDIT_LOG_DAYS`, `RETENTION_CHAT_DAYS`, `RETENTION_ASSESSMENT_DAYS`) so deployments can tune to their own policies.
  - When stricter external requirements apply, operators should lower the env-based retention durations and re-run cleanup jobs; if longer retention is mandated, envs can be increased up to infra/storage limits.

- **Day 2 – Scheduled job infrastructure**
  - Add a simple scheduler (e.g. `node-cron` or a hosted scheduler hitting a cleanup endpoint).
  - Implement a base job runner for periodic tasks.

- **Day 3 – Cleanup implementation**
  - Implement cleanup logic for:
    - Old audit logs.
    - Old chat messages beyond retention.
    - Orphaned/abandoned assessments if applicable.

- **Day 4 – Safety & observability**
  - Add safeguards:
    - Soft-delete or staged deletion for critical data if needed.
    - Basic logging/metrics around cleanup runs (how many records removed).

- **Day 5 – Documentation & review**
  - Document:
    - Retention decisions and rationale.
    - How to adjust retention windows.
  - Review for potential follow-up tasks (e.g. admin UI to configure retention).

**Week 9 – Implementation summary (done)**

- **Scheduler**: `server/src/jobs/scheduler.ts` – `registerJob({ name, intervalMs, run })`, `startJobs()`. Jobs run once on startup and then on `intervalMs` (setInterval). Logs start, completion, duration, and errors.
- **Retention job**: `server/src/jobs/retentionCleanupJob.ts` – `getRetentionConfig()` reads `RETENTION_AUDIT_LOG_DAYS` (default 540), `RETENTION_CHAT_DAYS` (default 360), `RETENTION_ASSESSMENT_DAYS` (default 720). `runRetentionCleanup()` deletes: (1) `AuditLog` where `createdAt` &lt; cutoff, (2) `ChatSession` where `lastMessageAt` or `createdAt` &lt; cutoff (cascade deletes messages), (3) `Assessment` where `completedAt` or `createdAt` &lt; cutoff (cascade to Answer/Score). If `RETENTION_CLEANUP_ENABLED=false` (or `0`), the job no-ops and logs once.
- **Registration**: `server/src/jobs/registerJobs.ts` registers the retention job with interval 24h. `server.ts` imports it and calls `startJobs()` after routes.
- **Runbook**: To **disable** cleanup: set `RETENTION_CLEANUP_ENABLED=false`. To **tune retention**: set `RETENTION_AUDIT_LOG_DAYS`, `RETENTION_CHAT_DAYS`, `RETENTION_ASSESSMENT_DAYS` (days). To **run cleanup manually** (e.g. from a script): import `runRetentionCleanup` from `./jobs/retentionCleanupJob` and `await runRetentionCleanup()`. Deletions are **hard deletes**; no soft-delete in this implementation. Logs report per-run counts (auditLogs, chatSessions, assessments deleted).
