# Railway Server: Push & Redeploy Guide

Step-by-step guide for pushing and redeploying the Complyx server on Railway (including migrations, Redis, and optional seeds).

---

## 1. Prepare locally

- Commit and push your server changes (including any new migrations under `server/prisma/migrations/`).
- Confirm the server builds: from repo root run `cd server && pnpm build` (no errors).

---

## 2. Confirm Railway link

- In the project directory run:
  ```bash
  railway status
  ```
- You should see **Project**: courageous-benevolence, **Environment**: production, **Service**: complyx_server.
- If not, run `railway link` and select the correct project, environment, and service.

---

## 2b. Set Root Directory (monorepo)

- In **Railway Dashboard** → your project → **complyx_server** service → **Settings** → **Source** (or **General**).
- Set **Root Directory** to `server` so Railway builds and runs from the `server/` folder (where `package.json` has the `start` script and `railway.json` defines the start command).
- Without this, Railpack builds from the repo root and fails with "No start command was found".

---

## 3. Set the start command (migrations on deploy)

- In **Railway Dashboard** → your project → **complyx_server** service → **Settings** (or **Variables** section).
- Find **Start Command** / **Custom start command**.
- Set it to:
  ```bash
  pnpm start:prod
  ```
- Save.

This runs `prisma generate` → `prisma migrate deploy` → `node dist/server.js` on every deploy so new migrations (assessment, telemetry, etc.) are applied automatically.

---

## 4. Ensure PostgreSQL and Redis

- **PostgreSQL**: In the same Railway project, confirm a Postgres service exists and is running. The server gets `DATABASE_URL` from it (e.g. `${{Postgres.DATABASE_URL}}`).
- **Redis**: If you have a Redis service, in the **server** service **Variables** add or confirm:
  ```bash
  REDIS_URL=${{Redis.REDIS_URL}}
  ```
  (Use the exact variable name Railway shows for your Redis service.) If you don’t use Redis yet, you can skip and add later.

---

## 5. (Optional) Run migrations once before first deploy

- If you prefer to apply migrations manually once instead of using `start:prod`:
  ```bash
  railway run pnpm exec prisma migrate deploy
  ```
- Then you can keep Start Command as `pnpm start`. For ongoing redeploys, using `pnpm start:prod` is simpler.

---

## 6. Deploy / redeploy

- **Option A – Deploy from Git**  
  Push to the branch connected to Railway; Railway will build and redeploy the server automatically.

- **Option B – Deploy from CLI**  
  From the repo root (with `railway link` pointing at complyx_server):
  ```bash
  railway up
  ```
  Or from the server directory:
  ```bash
  cd server && railway up
  ```

---

## 7. Check the deployment

- In Railway: **Deployments** → latest deployment → **View Logs**. Look for:
  - Successful build.
  - No Prisma/migration errors (e.g. “Applying migration…” then server start).
  - Server listening (e.g. on `PORT`).
- Hit your server’s health endpoint (e.g. `https://your-app.railway.app/health`) to confirm it’s up.

---

## 8. (Optional) Seed data after first successful deploy

- **Assessment question bank** (for in-chat assessments):
  ```bash
  railway run pnpm db:seed:assessment
  ```
- **Auth roles / initial users** (if needed):
  ```bash
  railway run pnpm db:seed:auth
  ```

---

## 9. Future redeploys

- Push to the linked branch (or run `railway up`).
- With Start Command = `pnpm start:prod`, new migrations will run on each deploy; no extra steps unless you add new env vars or new services (e.g. Redis) later.

---

## Quick checklist

| # | Step |
|---|------|
| 1 | Commit/push and ensure `pnpm build` passes in `server/`. |
| 2 | Run `railway status` and confirm correct project/service. |
| 3 | Set Start Command to `pnpm start:prod` in Railway. |
| 4 | Confirm Postgres + Redis (and `REDIS_URL`) in the project. |
| 5 | Deploy via Git push or `railway up`. |
| 6 | Check deployment logs and `/health`. |
| 7 | Optionally run `db:seed:assessment` (and `db:seed:auth`) once. |
