# Jasonex AI — Social Media Autopilot

Full-stack MVP scaffold for an AI content automation platform.

## Progress notes (backend + Supabase wiring pass)

**Wired to the live FastAPI backend (`apps/api`) — no more mock data:**
- `/login` — real signup + login (JWT), toggle between the two modes
- `/dashboard` (layout) — route is now protected; redirects to `/login` if not authenticated
- `/dashboard` (main) — real user name/plan/credits, real videos, real analytics overview
- `/dashboard/create-video` — actually calls `POST /api/videos` to generate a video
- `/dashboard/videos` — real video list, filters, and status counts from the API
- `/dashboard/analytics` — real `GET /api/analytics/overview` data
- Sidebar / Topbar — show the logged-in user's real name, plan, and credit usage, with a working logout

**All 15 pages are now wired to the live FastAPI backend — no mock data left in the dashboard:**
- `/login` — real signup + login (JWT), toggle between the two modes
- `/dashboard` (layout) — protected route; redirects to `/login` if not authenticated
- `/dashboard` (main) — real user name/plan/credits, real videos, real analytics overview
- `/dashboard/create-video` — calls `POST /api/videos` to generate a video
- `/dashboard/content-series` — calls `POST /api/content-series`
- `/dashboard/calendar` — real events from `GET /api/calendar/events`, add-event form posts to the API
- `/dashboard/videos` — real video list, filters, and status counts
- `/dashboard/editor` — loads a real video by id, scene edit calls the API
- `/dashboard/approval` — real pending-approval queue, approve/request changes/reject act on the API
- `/dashboard/analytics` — real `GET /api/analytics/overview` data
- `/dashboard/accounts` — real connected accounts, connect/disconnect call the API
- `/dashboard/brand-kit` — loads/saves via `GET`/`PUT /api/brand-kit`
- `/dashboard/ai-autopilot` — real config via `GET`/`PUT /api/ai-autopilot/config`, run-now button
- `/dashboard/ai-tools` — chat calls `POST /api/ai-tools/chat`
- `/dashboard/settings` — real subscription/plan data, upgrade calls `POST /api/subscriptions/upgrade`
- Sidebar / Topbar — show the logged-in user's real name, plan, and credit usage, with a working logout

`apps/web/lib/mock-data.ts` is no longer imported by any page; it's kept only for the shared
`VideoStatus` type used by `StatusBadge`.

Verified: `npx tsc --noEmit` and `npm run build` both pass clean (all 18 routes compile), and the
FastAPI app boots and registers all 36 routes with the default SQLite database.

## Supabase database

The backend now works out of the box with Supabase Postgres:
1. Create a project at supabase.com.
2. Project Settings → Database → Connection string → "Transaction pooler" (port 6543).
3. In `apps/api/.env`, set:
   ```
   DATABASE_URL=postgresql+psycopg://postgres.xxxxxxxx:[email protected]:6543/postgres
   ```
4. Start the API as usual (`uvicorn app.main:app --reload`) — on startup it runs
   `Base.metadata.create_all`, which creates all tables in your Supabase database automatically.

`SUPABASE_URL` / `SUPABASE_ANON_KEY` / `SUPABASE_SERVICE_ROLE_KEY` are also available in
`apps/api/app/core/config.py` for when you want to call Supabase Auth/Storage/Realtime directly
(not required for the current JWT-based auth, which works standalone against `DATABASE_URL`).

Frontend just needs `apps/web/.env.local` → `NEXT_PUBLIC_API_URL=http://localhost:8000` (or your
deployed API URL).



## Stack
- Web: Next.js 15 + React + TypeScript + Tailwind CSS
- API: FastAPI + SQLAlchemy + PostgreSQL
- Queue: Redis
- Worker: Python background worker scaffold
- Video: FFmpeg service adapter scaffold
- Auth: Google/phone OTP interfaces (provider wiring required)
- Billing: Stripe/Lemon Squeezy adapter interfaces (keys required)
- Publishing: YouTube/TikTok/Instagram adapter interfaces

## Run locally

### 1. Infrastructure
```bash
docker compose -f infra/docker-compose.yml up -d postgres redis
```

### 2. API
```bash
cd apps/api
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

### 3. Web
```bash
cd apps/web
npm install
npm run dev
```

Open http://localhost:3000.

### 4. Worker
```bash
cd apps/worker
pip install -r requirements.txt
python worker.py
```

## Important
This package is a production-oriented MVP scaffold. External credentials are intentionally not included. Add your API keys and OAuth credentials in environment variables before enabling real provider calls.

Free plan: 2 videos. After the limit is reached, the API returns HTTP 402 until a paid subscription is active.
# company-os-system
