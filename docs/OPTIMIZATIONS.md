# Optimizations Report – Database, History, Error Handling & Logging

## Overview

This document captures recommended optimizations for the Complyx platform, covering database, persistence, security, error handling, logging, scroll smoothness, and loading UX for both web and mobile views.

---

## 1. Chat History Persistence

| Aspect | Details |
|--------|---------|
| **Overview** | Chat messages are stored only in `localStorage` via Zustand. No server-side persistence exists. |
| **Work to be done** | Add `ChatSession` and `ChatMessage` tables; create save/load APIs; wire chat flow to persist messages for logged-in users. |
| **Impact** | History survives device changes and browser data loss; cross-device access; search, audit, and analytics. |

---

## 2. Redis Utilization

| Aspect | Details |
|--------|---------|
| **Overview** | Redis is configured (Docker, env) but not used in application code. |
| **Work to be done** | Add Redis client; implement token blacklist for logout; cache read-heavy data (questions, categories, dashboard); define TTLs and invalidation. |
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
| **Overview** | Logout only clears tokens on the client; no server-side revocation. |
| **Work to be done** | Implement token blacklist (e.g. Redis) with TTL; check blacklist in auth middleware; add logout endpoint that blacklists tokens. |
| **Impact** | Logout actually invalidates tokens; ability to revoke compromised tokens. |

---

## 6. Data Retention Policy

| Aspect | Details |
|--------|---------|
| **Overview** | No defined retention or cleanup for `AuditLog`, chat history, or old assessments. |
| **Work to be done** | Define retention rules; add scheduled jobs (e.g. `node-cron`) for cleanup; document policy. |
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

2. **Phase 2 (observability)**  
   Error handling (7), logging – web (8), logging – mobile (9).

3. **Phase 3 (UX)**  
   Loading UX (11).

4. **Phase 4 (security)**  
   Token storage (4), session revocation (5), Redis (2).

5. **Phase 5 (features & operations)**  
   Chat history persistence (1), data retention (6).
