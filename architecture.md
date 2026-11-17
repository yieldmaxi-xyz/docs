# System Architecture

## Overview

Crypto Yield Maxi is a monorepo with two deployable applications:

- `backend/`: Express 4 API, node-cron worker, Sequelize models, exchange clients, sync services
- `frontend/`: Next.js App Router application (Next 16, React 19)

Shared infrastructure:

- MySQL 8 for durable data
- Redis for distributed cache, locks and scanner snapshots
- PM2 for process management (separate API and cron worker)
- Telegram for operational alerts and hourly reports

## Active Data Domains

- Delta-neutral / funding extraction (same exchange and cross exchange)
- Cross-exchange arbitrage (two-leg scanner)
- Lending pools
- Vault opportunities
- Airdrops
- Exchange metadata
- RWA asset classification (shared symbol registry)
- Curated exchange guide content (frontend-owned)

## High-Level Flow

```text
External APIs / DefiLlama / venue metadata
  -> exchange clients + sync services / fetchers
  -> MySQL (current rows + history/snapshots)
  -> Express API (Redis + in-process caches)
  -> Next.js pages
```

## Backend Structure

| Path | Responsibility |
| --- | --- |
| `backend/src/app.ts` | Express middleware, CORS, rate limiting, cache headers, route mounting |
| `backend/src/server.ts` | API entry point; warms the arbitrage snapshot before listening |
| `backend/src/worker.ts` | Cron entry point; runs one instance only |
| `backend/src/routes/` | Route definitions |
| `backend/src/controllers/` | HTTP request/response handling |
| `backend/src/services/` | Sync services, aggregation, scanner engine, cache-aware queries |
| `backend/src/exchanges/` | One client per venue and the registry in `index.ts` |
| `backend/src/services/vaults/` | Dedicated vault fetchers |
| `backend/src/config/` | Env config, filters, periods, cache keys, Redis, DB |
| `backend/src/models/` | Sequelize models/tables |
| `backend/src/scripts/` | Manual syncs, migrations, backfills, admin tools |
| `backend/src/cron/` | Cron schedule and hourly funding snapshot helper |

## Active Cron Jobs

Defined in [backend/src/cron/index.ts](../backend/src/cron/index.ts). All schedules are UTC (`TZ=UTC` in PM2).

| Schedule | Job | Purpose |
| --- | --- | --- |
| `*/5 * * * *` | Arbitrage scanner | Full venue fetch, pair build, snapshot publish, query-cache warmup |
| `*/10 * * * *` | Scanner staleness monitor | Telegram alert when the scanner has not produced fresh data for 15 minutes |
| `3 * * * *` | Delta-neutral sync | Refresh current 8h rows, snapshot history, aggregate 24h/7d/30d plus 90d/180d (90d/180d throttled to every 6h) |
| `7 * * * *` | Lending sync | Refresh lending pools from DefiLlama |
| `9 * * * *` | Vault sync | Refresh vaults from dedicated fetchers + DefiLlama |
| `10 * * * *` | Top opportunities | Telegram/reporting only |
| `12 * * * *`, `42 * * * *` | History snapshot guard | Fill a missing or partial hourly `funding_extractor_history` slot |
| `15 * * * *` | Hourly summary | Telegram/reporting only |
| `50 * * * *` | Delta data health | Alert on history gaps / slow aggregation |
| `30 2 * * *` | Database backup | Dump to `BACKUP_DIR` |
| `30 3 * * *` | Retention cleanup | Prune history/snapshots per retention settings |
| `45 4 * * *` | History self-heal | Rebuild the last 48h of hourly history from venue APIs |

The scanner also has two independent guards:

- an in-process flag plus a Redis distributed lock prevent overlapping scans across the API/worker processes
- a background refresh keeps each API worker's RAM snapshot current without blocking requests

## Main Backend Tables

| Table | Purpose |
| --- | --- |
| `funding_extractor` | Current delta-neutral rows per symbol/exchange/period (`8h`, `24h`, `7d`, `30d`, `90d`, `180d`) |
| `funding_extractor_history` | Hourly funding history used for aggregation and charts |
| `delta_cross_snapshots` | Qualified hourly cross-exchange routes (spot venue + perp venue) |
| `delta_cross_period_summaries` | Pre-aggregated cross-exchange list rows per period/route |
| `funding_rate_snapshots` | Arbitrage scanner hourly funding-rate buckets |
| `lending_pools` | Normalized lending pools |
| `vaults` | Normalized vault opportunities |
| `rwa_assets` | Shared, symbol-keyed RWA/stock/commodity/index/FX registry |
| `exchange_fees` | Per-venue fee metadata used by APR/net-APR calculations |
| `ref_links` | Referral/protocol links keyed by slug and type (`exchange`, `lending`, `vault`, `airdrop`) |
| `airdrops` | Curated airdrop records |

Legacy arbitrage tables (`arbitrage_opportunities`, `arbitrage_history`) were removed on 2026-11-09.

## Frontend Data Usage

| Route | Data source |
| --- | --- |
| `/` | Static content (including the `380+` exchange stat) |
| `/delta-neutral` | `/api/funding-extractor`, `/api/funding-extractor/history` |
| `/delta-neutral?mode=cross` | `/api/funding-extractor/cross`, `/api/funding-extractor/cross/history`, `/api/exchanges` |
| `/arbitrage` | `/api/arbitrage/opportunities`, `/api/arbitrage/sync-status`, `/api/exchanges` |
| `/lending` | `/api/lending` |
| `/vaults` | `/api/vaults`, `/api/vaults/history` |
| `/airdrop` | `/api/airdrop` |
| `/guide` | Curated frontend lists in [frontend/lib/exchanges.ts](../frontend/lib/exchanges.ts) + `/api/home/exchange-ref-links` |
| `/health` | Frontend route handler; reads `release.json`, no backend call |

Server-side fetches call the backend directly (`INTERNAL_API_URL`, default `http://127.0.0.1:1000`) to avoid public edge/WAF routing. In the browser, requests use relative `/api/*` paths rewritten to the backend by Next.js, unless `NEXT_PUBLIC_API_URL` points to an external API host.

## Important Current Design Decisions

- **Exchange blacklist:** `BLACKLIST_EXCHANGES` defaults to `MEXC,HTX`. Blacklisted venues are excluded from ingestion, the scanner snapshot, `/api/exchanges` and delta-neutral results.
- **Delta-neutral coverage:** Same-exchange sync uses the curated `DELTA_SYNC_EXCHANGE_NAMES` list. Cross-exchange adds a curated perp-only short-leg list (`DELTA_CROSS_PERP_EXCHANGES`).
- **RWA classification:** `rwa_assets` is the authoritative runtime registry. Venue metadata can teach new symbols; explicit crypto contracts always win to avoid ticker collisions.
- **Arbitrage modes:** `mode=funding` returns rows with positive funding spread, `mode=basis` returns positive basis rows, `mode=realized` requires positive 24h or 7d realized funding arbitrage.
- **Cross-exchange aggregation:** List requests read `delta_cross_period_summaries`; raw hourly snapshots are kept for charts (180-day retention). Coverage is reported per row and long-period averages only use qualified hours.
- **Freshness guards:** Delta periods never serve rows older than the period-specific staleness window; scanner health reports stale data after 15 minutes.
- **Caching:** Redis is authoritative when connected. In-process memory is only a fallback when Redis is unavailable or a write failed.
- **Rate limiting:** General API traffic is limited per client IP (30 requests / 5 seconds, 30-second block). `/api/health` is mounted before the limiter.
- **Proxy awareness:** `TRUST_PROXY` plus `CF-Connecting-IP` / `X-Real-IP` / `X-Forwarded-For` handling keeps client IPs accurate behind Nginx/Cloudflare.
