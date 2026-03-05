# Mobile & Client Logging and Error Visibility (Short-term)

This document focuses on seeing errors and logs from mobile devices (and all clients) by sending them to **your own backend**—remote logger transport plus an error-reporting endpoint. A long-term option (e.g. Sentry) can be added later.

---

## Phase 1 – Remote Logging & Error Reporting (Weeks 1–2)

**Goal:** Send client logs and errors to your API and view them in one place (no third-party yet).

### Week 1 – Backend & Transport

- **Day 1 – Error-reporting endpoint**
  - Add `POST /api/telemetry/error` (or `/api/errors`) with body: `{ message, stack?, code?, url, userAgent?, userId?, level? }`.
  - Auth: optional (e.g. require auth for userId, or allow anonymous with rate limit).
  - Store in DB (e.g. `ClientError` table) or append to a log file; return 204.

- **Day 2 – Log-ingestion endpoint**
  - Add `POST /api/telemetry/log` accepting batches: `{ entries: [{ level, message, timestamp, context? }] }`.
  - Optional: persist in DB (e.g. `ClientLog` table with level, message, timestamp, JSON context) or stream to your logging pipeline.
  - Rate limit by IP or user (e.g. 100/min) to avoid abuse.

- **Day 3 – Wire logger to remote transport**
  - In your existing client logger, add a “remote” transport that:
    - Buffers recent log entries (e.g. last N or last 30s).
    - On `error` (and optionally `warn`), flush immediately; otherwise flush on interval or on page unload (e.g. `sendBeacon` or fetch to `/api/telemetry/log`).
  - Ensure mobile context (e.g. `isMobile`, viewport, connection) is included in payload so you can filter by device later.

- **Day 4 – Wire ErrorBoundary and API errors**
  - In `ErrorBoundary`, on `componentDidCatch`, call the logger (so the remote transport can send it) and/or call the error endpoint directly with message, componentStack, and context.
  - In your API client wrapper (e.g. where you handle non-2xx), on error send a minimal payload to `POST /api/telemetry/error` (message, code, url) so mobile API failures are visible.

- **Day 5 – Basic “view errors” surface**
  - Simple admin or internal page (or script) that lists recent rows from `ClientError` (and optionally recent `ClientLog`). Filter by date, level, and if you stored it, “mobile” (e.g. userAgent or `context.isMobile`). No need for fancy UI yet.

### Week 1 – Implementation Summary (Done)

- **Schema**: `ClientError` and `ClientLog` models in Prisma; migrations applied.
- **Server**: `telemetryController.ts` with `reportError` (POST /api/telemetry/error), `ingestLog` (POST /api/telemetry/log), `getErrors` (GET /api/telemetry/errors?limit=&level=). Routes in `telemetry.ts` and registered under `/api/telemetry`.
- **Client logger**: Remote transport in `logger.ts` – buffer, flush on error/warn or every 30s and on pagehide/beforeunload; sends to `/api/telemetry/log` with mobile context when `NEXT_PUBLIC_ENABLE_REMOTE_LOGGING=true`.
- **Error reporting**: `telemetryApi.reportClientError()`; used in `ErrorBoundary` and in `client.ts` on API non-2xx.
- **View surface**: `app/admin/client-errors/page.tsx` – lists recent client errors with level filter.

---

## Phase 2 – Harden & Observe (Week 3)

- **Day 1 – Sampling and safety**
  - Add sampling for non-error logs (e.g. send only 10% of `info` in production, 100% of `error`).
  - Sanitize payloads (no passwords, tokens, PII in message/context); truncate long stacks.

- **Day 2 – Retention and cleanup**
  - Define retention for `ClientError` / `ClientLog` (e.g. 30 days); add a small job or cron to delete older rows so the table doesn’t grow forever.

- **Day 3 – Use it from mobile**
  - Test from real devices: trigger an error and a few log levels, confirm they appear in your DB and in the “view errors” list. Document how to filter “mobile” (e.g. by userAgent or context flag).

**Filtering by mobile**
- **ClientError**: Each row has `userAgent`. To find mobile-reported errors: filter where `userAgent` contains typical mobile identifiers (e.g. `Mobile`, `Android`, `iPhone`, `iPad`, `webOS`). The admin page at `/admin/client-errors` shows `userAgent`; you can add a “Mobile” filter in the UI or query the API with a custom param later.
- **ClientLog**: Log entries are sent with `context` that includes `isMobile` (boolean, from `window.matchMedia('(max-width: 768px)')`). When storing in DB, `context` is JSON—filter rows where `context->>'isMobile' = 'true'` (PostgreSQL) or inspect the JSON in your view layer to show only mobile-origin logs.

- **Day 4 – Alerts (optional)**
  - If you have an alert channel (e.g. Slack, email), add a simple rule: when `ClientError` count in last 15 min > threshold, send a notification. Improves visibility for mobile errors without opening the dashboard.

- **Day 5 – Document**
  - Short doc or section in `OPTIMIZATIONS.md`: what endpoints exist, what’s sent, retention, how to view errors and logs (including from mobile).

### Phase 2 – Implementation Summary (Done)

- **Sampling and safety** (`client/lib/utils/logger.ts`): In production, only 10% of `info`/`debug` are sent to the remote transport; 100% of `error`/`warn`. Added `sanitizeForRemote` / `sanitizeContext` to strip sensitive keys (password, token, etc.) and truncate long strings before sending.
- **Retention** (`server/src/jobs/retentionCleanupJob.ts`): `ClientError` and `ClientLog` are deleted when older than 30 days (configurable via `RETENTION_CLIENT_ERROR_DAYS` and `RETENTION_CLIENT_LOG_DAYS`). Run by the same 24h retention job.
- **Mobile filtering**: Documented in this file under “Filtering by mobile” (userAgent for errors, context.isMobile for logs).
- **Alerts** (`server/src/jobs/telemetryAlertJob.ts`): Job runs every 15 min; if `ClientError` count in last 15 min exceeds `TELEMETRY_ALERT_THRESHOLD` (default 10), logs a warning and optionally POSTs to `TELEMETRY_ALERT_WEBHOOK_URL` (e.g. Slack). Registered in `registerJobs.ts`.
- **Documentation**: Section 12 in `OPTIMIZATIONS.md` describes endpoints, payloads, retention, view, and alert env vars.

---

## Summary

| Phase | Focus | Outcome |
|-------|--------|--------|
| **1 (W1–2)** | Your API + logger transport + error endpoint | See errors and logs from mobile in your own DB and a simple “view” page. |
| **2 (W3)** | Sampling, retention, testing, optional alerts | Safe, sustainable backend logging and clear “mobile” visibility. |

**Remaining short-term (done):** Rate limiting for `/api/telemetry` (100/min per IP via `telemetryLimiter` in `server`); admin page filters for date range (`from`/`to` query) and “Mobile only” (`mobile=true`, filters by userAgent patterns). See `GET /api/telemetry/errors?from=&to=&mobile=&level=&limit=`.

**Testing:** Run `pnpm test:telemetry-api` in the server directory. The script `server/src/scripts/testTelemetryApi.ts` exercises `reportError` (400 when message missing, 204 when valid), `ingestLog` (204 for empty or valid entries), and `getErrors` (200 with errors array, with optional level/mobile filters). No server process required; uses DB only.

**Later:** A long-term option (e.g. Sentry) for grouping, replay, and richer tooling can be added once this short-term setup is in place.
