# Deployment Guide

## Requirements

- Node.js 20+
- pnpm 10.26.2 (repo uses `packageManager: pnpm@10.26.2`)
- MySQL 8+
- Redis
- PM2
- reverse proxy such as Nginx or a managed platform equivalent

Enable pnpm on the server if needed:

```bash
corepack enable
corepack prepare pnpm@10.26.2 --activate
# or: npm install -g pnpm@10.26.2
```

## Backend Environment

Start from `backend/.env.example`. Minimum production example:

```env
NODE_ENV=production
PORT=1000
TZ=UTC

DB_HOST=localhost
DB_PORT=3306
DB_NAME=crypto_yield_maxi
DB_USER=app_user
DB_PASSWORD=secure_password

REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

LOG_LEVEL=info
TRUST_PROXY=1
```

### Important backend variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `TRUST_PROXY` | `false` | Number of proxy hops, or `true`/`false`. Use `1` behind Nginx/Cloudflare → Next → API |
| `BLACKLIST_EXCHANGES` | `MEXC,HTX` | Venues excluded from ingestion and API output; empty = disable blacklist |
| `DELTA_MIN_SPOT_VOLUME` | `100000` | Delta-neutral spot volume floor |
| `DELTA_MIN_PERP_VOLUME` | `200000` | Delta-neutral perp volume floor (`.env.example` uses `500000`) |
| `DELTA_MIN_PERP_OI` | `300000` | Delta-neutral perp OI floor (`.env.example` uses `1000000`) |
| `DELTA_CROSS_MAX_BASIS_PCT` | `3` | Max absolute basis for cross routes |
| `DELTA_CROSS_TOP_ROUTES_PER_SYMBOL` | `0` | Route storage cap; `0` = store all |
| `DELTA_CROSS_PERP_EXCHANGES` | curated list | Perp-only venues for cross mode; empty = disable |
| `CROSS_DELTA_RETENTION_DAYS` | `180` | Raw cross snapshot retention |
| `CROSS_SUMMARY_RETENTION_DAYS` | `180` | Cross period-summary retention |
| `ARBITRAGE_MIN_PERP_VOLUME` | `200000` | Reporting threshold |
| `ARBITRAGE_MIN_PERP_OI` | `100000` | Reporting threshold |
| `DATA_RETENTION_YEARS` | `2` | Funding history retention |
| `BACKUP_DIR` | `./backups` | Database dump directory |
| `BACKUP_RETENTION_DAYS` | `7` | Backup rotation |
| `TELEGRAM_BOT_TOKEN` | — | Alert/report bot |
| `TELEGRAM_MONITOR_CHAT_ID` | — | Ops alerts |
| `TELEGRAM_TOP_CHAT_ID` | — | Top-opportunities channel |
| `ADMIN_TOKEN` | — | Required for `POST /api/arbitrage/sync` in production |

`TRUST_PROXY` notes:

- Behind Nginx/Cloudflare (one proxy hop): `TRUST_PROXY=1`.
- Directly exposed with no proxy: `TRUST_PROXY=false` is fine.
- A miscount breaks rate limiting because `req.ip` becomes the proxy address.

## Backend Deploy Steps

```bash
cd /var/www/crypto-yield-maxi/backend
git pull origin main
pnpm install --frozen-lockfile
pnpm build
pm2 reload ecosystem.config.cjs
pm2 save
```

PM2 runs two apps from `backend/ecosystem.config.cjs`:

| App | Script | Mode | Notes |
| --- | --- | --- | --- |
| `YIELD-MAXI-API` | `dist/server.js` | cluster, 1 instance | 1 GB memory limit, zero-downtime reload |
| `YIELD-MAXI-CRON` | `dist/worker.js` | fork, 1 instance | Must stay single to avoid duplicate cron runs; restarts daily at `05:30 UTC` |

Both use `TZ=UTC`. The API performs a bounded warmup of the arbitrage snapshot before accepting traffic.

## Database Maintenance

Run schema sync after deploy when models changed:

```bash
pnpm db:sync
```

Available migrations:

```bash
pnpm migrate:funding-range        # widen funding_extractor.funding_8h_pct
pnpm migrate:spot-fees            # add spot fee columns
pnpm migrate:indexes              # optimize DB indexes
pnpm migrate:fix-timezone         # fix history timezone drift
pnpm migrate:drop-legacy-arbitrage
pnpm migrate:cross-delta          # create delta_cross_snapshots
pnpm migrate:cross-summary        # create delta_cross_period_summaries
pnpm migrate:cross-final-apr      # fix cross final APR basis
pnpm migrate:cross-summary-prices # add cross summary price columns
pnpm migrate:funding-coverage     # add funding_extractor coverage columns
pnpm migrate:rwa-assets           # create + seed rwa_assets
```

Useful backfills and tools:

```bash
pnpm backfill:funding
pnpm backfill:hourly
pnpm backfill:cross-summary
pnpm protocol:add <type> <slug> <url>
pnpm protocol:list
pnpm protocol:remove <slug>
pnpm rwa:list
pnpm rwa:add -- NVDA stock
pnpm rwa:sync
pnpm rwa:audit
```

## Manual Syncs

```bash
pnpm sync:delta
pnpm sync:arbitrage
pnpm sync:lending
pnpm sync:vault
pnpm sync:all
```

## Frontend Environment

Start from `frontend/.env.example`:

```env
# Development only; production same-server should leave this empty/relative.
NEXT_PUBLIC_API_URL=http://localhost:1000

NEXT_PUBLIC_WALLET_BTC=...
NEXT_PUBLIC_WALLET_ETH=...
NEXT_PUBLIC_WALLET_SOL=...
NEXT_PUBLIC_WALLET_SUI=...

NEXT_PUBLIC_CONTACT_EMAIL=...
NEXT_PUBLIC_CONTACT_WEBSITE=https://yieldmaxi.xyz
NEXT_PUBLIC_CONTACT_X=https://x.com/yieldmaxi_xyz
NEXT_PUBLIC_CONTACT_TELEGRAM=https://t.me/yieldmaxi_xyz
```

`INTERNAL_API_URL` is set by `frontend/ecosystem.config.cjs` to `http://127.0.0.1:1000` for server-side fetches. Do **not** point `NEXT_PUBLIC_API_URL` at `http://localhost:1000` in production; browser requests would target the visitor's machine.

## Frontend Deploy Steps

```bash
cd /var/www/crypto-yield-maxi/frontend
git pull origin main
pnpm install --frozen-lockfile
pnpm build
pm2 reload ecosystem.config.cjs
pm2 save
```

Frontend PM2 app:

| App | Script | Port |
| --- | --- | --- |
| `YIELD-MAXI-FRONTEND` | `npm start` (`next start -H 127.0.0.1 -p 1001`) | `1001` |

Next.js rewrites `/api/:path*` to `INTERNAL_API_URL`. Security headers and report-only CSP are defined in `next.config.ts`.

## Health Checks

Backend:

```bash
curl -fsS http://127.0.0.1:1000/api/health
```

- Returns `200` with `status: ok | degraded | error`.
- `degraded` when scanner data is older than 15 minutes or Redis is unavailable.
- `500` only when the database check fails/times out.
- `/api/health` is excluded from the API rate limiter.

Frontend:

```bash
curl -fsS http://127.0.0.1:1001/health
```

- Returns `{ "component": "frontend", "release": "<sha>" }` when `release.json` exists.
- `unversioned` for local/legacy builds.

## Operational Notes

- Redis is required for distributed scanner locks and cross-process snapshots. If Redis is down, the scanner falls back to per-process behaviour and caches become process-local.
- The 5-minute scanner sync can legitimately take up to ~90 seconds per large venue on a cold start; per-venue timeouts are 15s for symbols and 90s for funding.
- `OKX` is typically the slowest upstream source; watch `[ArbitrageScanner] Slow query` and cron logs.
- Daily backup, retention cleanup and history self-heal run at `02:30`, `03:30`, and `04:45 UTC`.
- Scanner staleness alerts fire after 15 minutes without a fresh snapshot (three missed cycles).
