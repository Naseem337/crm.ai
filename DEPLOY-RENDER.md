# Deploying to Render

## What runs where

| Piece | Where | Why |
|---|---|---|
| builder (web app) | Render web service | Docker, port 3000 |
| realtime | Render web service | Docker, port 1999 |
| worker | Render background worker | Docker |
| Redis | Render Key Value | in `render.yaml` |
| **PostgreSQL** | **Timescale Cloud** | migrations need TimescaleDB + pgvector; Render Postgres has neither TimescaleDB |
| File storage | Cloudflare R2 or AWS S3 | Render has no object storage |

## Steps

1. Push this repo to **your** GitHub repo (Render deploys from Git).
2. Create a Timescale Cloud service (PostgreSQL 17/18). Copy the connection string.
3. Create an R2 or S3 bucket and an access key.
4. Render dashboard > New > Blueprint > select the repo. It reads `render.yaml`.
5. Fill the prompted variables:
   - `DATABASE_URL` (same value on builder and worker)
   - `BETTER_AUTH_URL` and `NEXT_PUBLIC_BUILDER_URL` = your public URL
   - `ENCRYPTION_KEY` = output of `openssl rand -hex 32` (same on builder and worker; never change after launch)
   - `BETTER_AUTH_SECRET`: copy the builder's generated value into the worker
   - `REALTIME_BROADCAST_SECRET`: copy the builder's generated value into realtime
   - `REALTIME_BUILDER_URL` = the builder's internal URL
   - `S3_*` values
6. First deploy runs migrations and seeding (`RUN_DB_MIGRATE` / `RUN_DB_SEED`). After it succeeds, set `RUN_DB_SEED` to `false`.
7. Add your custom domain on the builder service, then update `BETTER_AUTH_URL` and `NEXT_PUBLIC_BUILDER_URL`.

## Known gaps

- `REALTIME_PROXY_URL` must be a full URL (`http://...`); verify it after the first deploy.
- Other integrations (WhatsApp, Stripe, SMTP, AI keys) are set as extra env vars; see `.env.example`.
- Unverified: this blueprint has not been deployed yet.
