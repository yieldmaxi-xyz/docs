# API Reference

Base URL:

- development: `http://localhost:1000`
- production: backend domain for your deployment (or relative `/api/*` through the Next.js rewrite)

## Conventions

- All endpoints are public `GET` routes unless noted.
- Unknown `/api/*` routes return JSON `404 { "error": "Not Found" }`.
- Public `GET` responses send `Cache-Control: public, max-age=60, s-maxage=60, stale-while-revalidate=60`. Error responses (`>= 400`) are forced to `no-store`.
- General API traffic is rate limited per client IP: **30 requests / 5 seconds**, then a **30-second block** with HTTP `429`, `Retry-After`, and `retry_after`.
- `/api/health` is mounted before the limiter and is safe for probes.
- Numeric filters accept empty strings as "not set".
- Comma-separated list filters accept both `?key=a,b` and repeated `?key=a&key=b`.

## Routes

### `GET /api/health`

Connectivity and data-freshness check.

Response:

```json
{
  "status": "ok | degraded | error",
  "timestamp": 1760000000000,
  "database": "connected",
  "redis": "connected | not available (caching disabled)",
  "scanner": {
    "generated_at": "2026-11-17T10:00:00.000Z",
    "age_secs": 42,
    "stale": false
  }
}
```

Notes:

- HTTP `200` is returned even when degraded, so PM2/load balancers do not restart the process for stale data.
- `degraded` means scanner data is older than 15 minutes or Redis is unavailable.
- `500` is returned only when the database connectivity check fails (3-second timeout).

### `GET /api/funding-extractor`

Returns delta-neutral / funding extraction opportunities for one aggregation period.

Query params:

| Param | Values | Notes |
| --- | --- | --- |
| `period` | `8h`, `24h`, `7d`, `30d`, `90d`, `180d` | Defaults to `90d`; invalid values return `400` |
| `exchanges` | comma-separated venue names | Optional |
| `min_rate` | number | Deprecated: filters the canonical `funding_8h_pct` field (cumulative for periods > 8h) |
| `min_apr` | number | Recommended; filters `final_apr_pct` |
| `min_gross_apr` | number | Filters annualized funding before costs |

Response:

```json
{
  "data": [
    {
      "symbol": "BTC",
      "exchange": "Hyperliquid",
      "vol_spot_ms": 123.4,
      "oi_perp_ms": 456.7,
      "funding_8h_pct": 0.0123,
      "apr_gross_pct": 13.5,
      "fee_spot_pct": 0.05,
      "fee_perp_pct": 0.03,
      "spread_spot_pct": 0.05,
      "spread_perp_pct": 0.03,
      "price_diff_pct": 0.1,
      "final_apr_pct": 12.9,
      "time_cost_days": 3,
      "coverage": {
        "samples": 720,
        "oldest": "2026-08-19T00:00:00.000Z",
        "newest": "2026-11-17T10:00:00.000Z",
        "coverage_pct": 100
      },
      "ref_link": "https://..."
    }
  ],
  "lastUpdated": "2026-11-17T10:03:00.000Z",
  "oldestDataAt": "2026-11-17T09:00:00.000Z"
}
```

Behavior:

- Rows are sorted by `final_apr_pct` descending.
- Spot volume and perp OI thresholds come from `DELTA_MIN_SPOT_VOLUME` / `DELTA_MIN_PERP_OI` (no per-request override on this endpoint).
- Stale period rows (older than the period-specific freshness window) are excluded.
- `coverage` is `null` for live `8h` rows and populated for aggregated periods.
- Blacklisted exchanges are excluded.

### `GET /api/funding-extractor/history`

APR history for one same-exchange delta-neutral route (chart popup).

Query params:

- `symbol` (required)
- `exchange` (required)
- `days` (optional, default `90`, clamped to `1..365`)
- `period` (optional fallback: `7d`, `30d`, `90d`, `180d`; `8h`/`24h` map to 7 days)

Returns an array of points:

```json
[{ "period_start": "2026-11-01T00:00:00.000Z", "final_apr_pct": 12.34 }]
```

Missing `symbol` or `exchange` returns `400`.

### `GET /api/funding-extractor/cross`

Cross-exchange delta-neutral: spot on venue A + short perp on venue B for the same canonical asset.

Query params:

| Param | Notes |
| --- | --- |
| `period` | `1d` (default), `7d`, `30d`, `90d`, `180d`; invalid values fall back to `1d` |
| `routes` | Comma-separated route keys: `canonical_symbol-spot_exchange-perp_exchange` |
| `spot_exchange`, `perp_exchange` | Comma-separated venue filters |
| `symbol` | Search on symbol/canonical symbol |
| `min_spot_vol` | Defaults to `DELTA_MIN_SPOT_VOLUME` |
| `min_perp_oi` | Defaults to `DELTA_MIN_PERP_OI` |
| `max_basis` | Absolute basis filter; defaults to `DELTA_CROSS_MAX_BASIS_PCT` (3%) |
| `min_apr` | Defaults to `0` |
| `sort` | `final_apr` (default), `gross_apr`, `basis`, `funding`, `spot_volume`, `perp_oi`, `time_cost`, `symbol` |
| `desc` | `true` (default unless explicitly `false`) |
| `page` | Default `1` |
| `per_page` | Default `20`, max `200` |

Response:

```json
{
  "data": [
    {
      "symbol": "BTC",
      "canonical_symbol": "BTC",
      "spot_exchange": "Binance",
      "perp_exchange": "Hyperliquid",
      "basis_pct": 0.12,
      "spot_volume_usd": 9500000,
      "perp_oi_usd": 42000000,
      "funding_8h_pct": 0.011,
      "funding_window_pct": 0.99,
      "fee_spot_pct": 0.1,
      "fee_perp_pct": 0.06,
      "spread_spot_pct": 0.05,
      "spread_perp_pct": 0.03,
      "gross_apr_pct": 14.2,
      "final_apr_pct": 13.1,
      "time_cost_days": 2,
      "coverage": {
        "samples": 24,
        "oldest": "2026-11-16T11:00:00.000Z",
        "newest": "2026-11-17T10:00:00.000Z",
        "coverage_pct": 100
      }
    }
  ],
  "total": 1,
  "page": 1,
  "per_page": 20,
  "total_pages": 1,
  "telemetry": {
    "generated_at": "2026-11-17T10:00:00.000Z",
    "oldest_data_at": "2026-11-17T09:00:00.000Z"
  }
}
```

Behavior:

- Only routes where both spot and perp data are present and valid are returned.
- The list response reads the pre-aggregated `delta_cross_period_summaries` table; raw hourly `delta_cross_snapshots` are kept for charts.
- Long-period averages are **qualified-hours averages** (hours that passed liquidity/basis filters), not unconditional window averages.
- `funding_8h_pct` is the average 8h-equivalent rate; `funding_window_pct` is the estimated cumulative funding over the selected window (extrapolated when coverage < 100%).
- The API returns one row per symbol: the route with the best `final_apr_pct`.
- Storage keeps every qualified route per symbol per hour by default; `DELTA_CROSS_TOP_ROUTES_PER_SYMBOL` can cap it (`0` = no cap).
- Raw cross snapshots and period summaries are retained for **180 days** by default (`CROSS_DELTA_RETENTION_DAYS`, `CROSS_SUMMARY_RETENTION_DAYS`).

### `GET /api/funding-extractor/cross/history`

Historical Final APR for one cross-venue route.

Required: `symbol`, `spot_exchange`, `perp_exchange`

Optional:

- `days` (default `90`, clamped to `1..180`)
- `period` fallback: `7d`, `30d`, `90d`, `180d`; `1d`/`1h`/`8h`/`24h` map to 7 days

Missing required params return `400`.

### `GET /api/arbitrage/opportunities`

Two-leg arbitrage scanner across all non-blacklisted venues. This is the endpoint used by `/arbitrage`.

Query params:

| Param | Notes |
| --- | --- |
| `mode` | `funding` (positive funding spread), `basis` (positive price basis), `realized` (positive 24h or 7d realized funding arb) |
| `sort` | See list below |
| `desc` | `true` (default) / `false` |
| `page` | Default `1` |
| `per_page` | Default `50`, max `200` |
| `exchanges` | Multi-select venue filter. One venue matches either leg; multiple venues require both legs in the set |
| `anchors` | At least one leg must be an anchored venue |
| `exclude` | Ticker exclusion list |
| `ids` | Restrict to explicit opportunity IDs (watchlist view) |
| `long_exchange`, `short_exchange` | Directed leg filters |
| `ticker` | Symbol search across canonical and venue-native symbols |
| `min_vol`, `max_vol` | Pair minimum 24h volume |
| `min_oi`, `max_oi` | Pair minimum open interest |
| `min_apr`, `max_apr` | `net_apr` range |
| `min_in_spread`, `max_in_spread` | `basis_pct` range |

Supported `sort` keys:

- `net_apr`, `net_apr_24h`, `net_apr_7d`
- `fund_spread_apr` (API default when no sort is supplied and mode is not `basis`), `fund_spread`, `fund_spread_1h` / `fund_1h`
- `fund_arb_24h` / `fund_24h`, `fund_arb_7d` / `fund_7d`
- `basis_pct` (default for basis mode) / `in_pct`
- `est_fee` / `est_fee_usd` / `fee`
- `vol` / `volume`, `oi`, `time_cost` / `time_cost_hours`, `symbol`
- `apr` (falls back to `net_apr` or `funding_spread_apr`)

Rows without the sorted metric always sort last, including ascending sorts.

Response:

```json
{
  "data": [
    {
      "id": "BTC:Binance-Hyperliquid",
      "symbol": "BTC",
      "asset_type": "CRYPTO",
      "asset_class": "CRYPTO",
      "long_exchange": "Binance",
      "long_symbol": "BTCUSDT",
      "long_ask": 95000,
      "long_rate": 0.0001,
      "long_interval_h": 8,
      "short_exchange": "Hyperliquid",
      "short_symbol": "BTC",
      "short_bid": 95020,
      "short_rate": 0.00012,
      "short_interval_h": 1,
      "basis_pct": 0.021,
      "funding_spread": 0.00002,
      "funding_spread_1h_pct": 0.001,
      "funding_spread_apr": 8.76,
      "funding_arb_24h_pct": 0.2,
      "funding_arb_7d_pct": 1.1,
      "est_fee_pct": 0.12,
      "net_apr": 8.2,
      "pair_min_vol_24h": 500000000,
      "pair_min_oi": 1000000000,
      "time_cost_hours": 2
    }
  ],
  "total": 1,
  "page": 1,
  "per_page": 50,
  "total_pages": 1,
  "telemetry": {
    "generated_at": "2026-11-17T10:00:00.000Z",
    "cycle_secs": 300,
    "age_secs": 30,
    "next_update_at": "2026-11-17T10:05:00.000Z",
    "total_pairs": 1
  }
}
```

Notes:

- The snapshot is refreshed every 5 minutes by the cron worker.
- Query results are cached per cycle and common page-1 variants are warmed after each sync.
- Response includes `Server-Timing: arbitrage;dur=...`.
- On a cold start with no snapshot, the API returns an empty cycle immediately and triggers a background sync instead of blocking the request.

### `GET /api/arbitrage/sync-status`

Lightweight polling endpoint for the countdown badge.

```json
{
  "is_syncing": false,
  "generated_at": "2026-11-17T10:00:00.000Z",
  "next_update_at": "2026-11-17T10:05:00.000Z"
}
```

### `POST /api/arbitrage/sync`

Manual/on-demand full scanner sync.

- Requires `x-admin-token: $ADMIN_TOKEN` when `ADMIN_TOKEN` is configured.
- In production without `ADMIN_TOKEN`, the endpoint returns `403`.
- Returns `{ success, generated_at, total_pairs, active_venues }`.

### `GET /api/lending`

Returns lending pools grouped by asset category.

Query params: none.

Response:

```json
{
  "data": {
    "USDC": [
      {
        "pool_id": "...",
        "project": "Aave V3",
        "chain": "Ethereum",
        "symbol": "USDC",
        "tvl_usd": 123456789,
        "apy": 5.2,
        "apy_base": 3.1,
        "apy_reward": 2.1,
        "apy_30d": 4.8,
        "project_url": "https://...",
        "logo_url": "https://..."
      }
    ],
    "USDT": [],
    "BTC": [],
    "ETH": [],
    "SOL": [],
    "HYPE": []
  },
  "lastUpdated": "2026-11-17T10:07:00.000Z",
  "oldestLastUpdated": "2026-11-17T10:07:00.000Z"
}
```

Notes:

- Up to 50 pools per category, ordered by APY descending.
- Categories are `USDC`, `USDT`, `BTC`, `ETH`, `SOL`, `HYPE`; Pendle PT-HYPE pools are excluded from `HYPE`.
- `lastUpdated` is the newest underlying row; `oldestLastUpdated` is the oldest displayed row for conservative freshness badges.
- Protocol URLs prefer `ref_links` overrides, then the sync-resolved protocol URL.

### `GET /api/vaults`

Returns vault opportunities.

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "DL-123",
      "protocol": "Hyperliquid",
      "symbol": "USDC",
      "apy_current": 8.5,
      "tvl_usd": 12500000,
      "deposit_token": "USDC",
      "lockup_period": "None",
      "min_deposit": null,
      "max_deposit": null,
      "project_url": "https://...",
      "logo_url": "https://...",
      "last_updated": "2026-11-17T10:09:00.000Z"
    }
  ]
}
```

Rows are sorted by `apy_current` descending and use `ref_links` overrides when present.

### `GET /api/vaults/history`

APR history for the vault chart popup.

Query params:

- `id` (required); `DL-*` IDs are fetched live from DefiLlama
- `days` (optional, clamped to `730`; omitted = full available series)

Returns an array of `{ period_start, final_apr_pct }` points. Missing `id` returns `400`; unknown non-DefiLlama vault returns `404`.

### `GET /api/airdrop`

Returns active curated airdrop records sorted by type then name.

```json
[
  {
    "name": "Hyperliquid",
    "type": "perp_dex",
    "logo": "https://...",
    "description": "...",
    "how_to_earn": "...",
    "estimated_value": "high",
    "view_order": 1,
    "raised": "...",
    "backed_by": "[\"Backer A\",\"Backer B\"]",
    "timeline": "...",
    "ref_link": "https://..."
  }
]
```

`type` is one of `perp_dex`, `hip3`, `defi`. `estimated_value` is `low`, `medium`, or `high`.

### `GET /api/exchanges`

Returns the live exchange registry with metadata.

```json
{
  "exchanges": [
    {
      "name": "Hyperliquid",
      "abbr": "HL",
      "slug": "hyperliquid",
      "type": "dex",
      "nativePeriod": "1h",
      "hasSpot": true,
      "supportsDeltaNeutral": true
    }
  ]
}
```

Blacklisted venues are omitted. The list is cached for 12 hours.

### `GET /api/home/total-exchanges`

Returns the homepage exchange count.

```json
{ "count": 380 }
```

Implementation detail: counts distinct `ref_links.protocol_slug` values, excluding Apex entries, and falls back to `380` when empty/error. The homepage itself now renders the static `380+` stat without calling this endpoint.

### `GET /api/home/exchange-ref-links`

Returns exchange referral-link overrides keyed by slug.

```json
{ "binance": "https://...", "hyperliquid": "https://..." }
```

Used by `/guide` to override curated defaults.

## Removed / No Longer Used

- `GET /api/arbitrage` (legacy matrix API; replaced by `/api/arbitrage/opportunities`)
- `GET /api/arbitrage/markets` (raw market pairing endpoint)
- `mode=pair` / `mode=all` arbitrage flow
- `GET /api/home/teasers`
- Arbitrage table sync (legacy `arbitrage_opportunities`, `arbitrage_history` tables)
- Exchange-directory cron (the registry is built from clients; the guide is curated in frontend code)

## Operational Notes

- If the app is behind a reverse proxy or CDN, configure `TRUST_PROXY` correctly and preserve client IP headers so rate limiting is accurate.
- Redis is optional for startup but required for distributed locks and cross-process scanner snapshots. Without it, caches fall back to process memory and limiter fallback is per process.
- A `503` from the rate limiter means throttling infrastructure was unavailable; retry shortly.
