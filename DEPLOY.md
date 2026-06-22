# Deployment Guide

## What This App Is

A **pnpm monorepo** (built on Replit) with two parts:
- **Frontend**: `artifacts/caktus-portfolio/` — React + Vite + Tailwind SPA
- **Backend**: `artifacts/api-server/` — Express.js REST API + PostgreSQL (via Drizzle ORM)

Vercel handles the **frontend**. The **backend** needs a separate host (Railway, Render, Fly.io, etc.).

---

## Deploying the Frontend to Vercel

### Option A — Vercel Dashboard (Recommended)

1. Push this repo to GitHub
2. Go to [vercel.com](https://vercel.com) → **New Project** → import your repo
3. In the project settings, set:
   - **Framework Preset**: `Vite`
   - **Root Directory**: `artifacts/caktus-portfolio`
   - **Build Command**: `npm run build`
   - **Output Directory**: `dist`
   - **Install Command**: `npm install`
4. Add this environment variable:
   - `VITE_API_URL` = your backend URL (e.g. `https://your-api.railway.app`)
5. Click **Deploy**

> The `vercel.json` at the repo root handles SPA rewrites automatically.

### Option B — Vercel CLI

```bash
cd artifacts/caktus-portfolio
npm install
npx vercel --prod
```

---

## Deploying the Backend (Railway / Render / Fly.io)

The backend is a standard Express.js server. Deploy `artifacts/api-server/`.

### Required Environment Variables

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `SESSION_SECRET` | Random secret string (min 32 chars) |
| `PORT` | Port number (Railway/Render set this automatically) |
| `ALLOWED_ORIGINS` | Comma-separated list of your frontend URLs, e.g. `https://your-app.vercel.app` |
| `NODE_ENV` | Set to `production` |

### Optional Variables

| Variable | Description |
|---|---|
| `GCS_BUCKET_NAME` | Google Cloud Storage bucket for file uploads |
| `GOOGLE_SERVICE_ACCOUNT_KEY` | GCS service account JSON (base64 or JSON string) |
| `SMTP_HOST` / `SMTP_PORT` / `SMTP_USER` / `SMTP_PASS` | Email config for contact form |

### Database Setup

After deploying the API, run migrations:
```bash
cd lib/db
npx drizzle-kit push
```

Or run `drizzle-kit migrate` if you have migration files.

### Build & Start Commands (for Railway/Render)

- **Build**: `cd artifacts/api-server && npm install && npm run build`
- **Start**: `node artifacts/api-server/dist/index.mjs`

---

## Connecting Frontend to Backend

In your Vercel project's environment variables, set:
```
VITE_API_URL=https://your-backend-url.railway.app
```

Then in your frontend API calls, use `import.meta.env.VITE_API_URL` as the base URL.

---

## What Was Fixed (from Replit original)

1. **`vite.config.ts`** — Removed hard `PORT` and `BASE_PATH` env requirements that crashed the Vite build. Removed `@replit/vite-plugin-*` plugins (Replit-only).
2. **`artifacts/caktus-portfolio/package.json`** — Removed `@replit/vite-plugin-*` devDependencies and `@workspace/api-client-react` workspace reference (not needed for standalone frontend build).
3. **`artifacts/api-server/src/app.ts`** — Replaced `REPLIT_DOMAINS` CORS variable with standard `ALLOWED_ORIGINS`.
4. **`pnpm-workspace.yaml`** — Removed `minimumReleaseAge` (Replit-specific pnpm feature, not supported by standard pnpm).
5. **`vercel.json`** — Created with SPA rewrite rules.
