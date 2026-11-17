# Frontend Pages

## Current Routes

| Route | Purpose | Notes |
| --- | --- | --- |
| `/` | Landing page | Product overview, feature cards, static `380+` exchange stat |
| `/about` | About / support page | Project explanation, donation wallets, contact links |
| `/guide` | Guide + curated exchange directory | Curated exchange data + DB referral-link overrides |
| `/delta-neutral` | Delta Neutral opportunities | Same-exchange and cross-exchange modes, URL-synced period/venue |
| `/arbitrage` | Cross-exchange arbitrage | Two-leg scanner with server-side filter/sort/pagination |
| `/lending` | Lending pools grouped by asset | Client-side grouped sections and sticky side navigation |
| `/vaults` | Vault opportunities | Notice modal and APR history popup |
| `/airdrop` | Airdrop opportunities | Card presentation grouped by perp DEX / HIP-3 / DeFi |
| `/trending` | Placeholder page | Coming-soon screen |
| `/health` | Route handler (not a page) | Returns `{ component: "frontend", release: "<sha>" }` with `no-store` |

`/error` and `/not-found` are shared App Router states.

## Notable Current UX

### `/arbitrage`

- Two quick modes: **⚡ Price** (`mode=basis`) and **⚡ Funding** (`mode=funding`). The API also supports `mode=realized` via URL/API.
- Server-side filtering, sorting and pagination through `GET /api/arbitrage/opportunities`.
- Countdown and polling use `GET /api/arbitrage/sync-status` (5-minute cycle).
- Preferences persist in `localStorage` and sync with URL params (including back/forward navigation).
- Features:
  - ticker search
  - venue and anchor filters
  - advanced numeric thresholds (volume, OI, net APR, basis)
  - column visibility dropdown
  - watchlist (starred rows) persisted in `localStorage`
  - mobile two-deck bento cards
  - profit calculator modal per opportunity
  - error banner with retry
- The watchlist-only view sends explicit opportunity IDs, so an empty watchlist shows zero rows.

### `/delta-neutral`

- Mode pills: **Same exchange** and **Cross exchange**.
- Same-exchange periods: `7d`, `30d`, `90d`, `180d` (default `90d`).
- Cross-exchange periods: `1d`, `7d`, `30d`, `90d`, `180d` (default `1d`).
- Venue selector defaults to Hyperliquid; `ALL` removes the venue filter.
- Period/venue/mode are written to the URL and survive back/forward navigation.
- Same-exchange short periods poll hourly; 90d/180d use a 6-hour cycle. Cross-exchange follows a similar fast/slow split.
- Tables use dedicated mobile card layouts.
- APR history chart supports 7D / 30D / 90D / 180D. Cross-route charts pass the selected period when it is 7d or longer; for `1d` they fall back to the chart's default 90d window.
- Coverage meters show qualified-hour coverage for aggregated cross routes.

### `/lending`

- Fetches the grouped lending dataset once, then renders sections client-side.
- Core sections: USDC, USDT, HYPE, BTC, ETH, SOL; variants are merged into their parent section.
- Unknown asset keys from the API render as additional generic sections instead of being dropped.
- HYPE excludes Pendle pools.
- Sticky side navigation and dedicated mobile presentation.
- Polling cycle: 1 hour.

### `/vaults`

- Notice content is i18n-driven; `Wasabi Perps` maps to the `Wasabi` notice key.
- Vaults without detailed notice copy still show a generic notice rather than a dead action.
- APR history popup: `DL-*` vaults use the DefiLlama series with the 7D / 30D / 90D / 180D selector; DB-only vaults render their fixed 7d/30d/current points without the selector.
- Polling cycle: 1 hour; the freshness badge uses the newest vault timestamp.

### `/airdrop`

- Fetches `/api/airdrop` client-side.
- Groups cards by `perp_dex`, `hip3`, and `defi`.
- Shows estimated value, funding/backers, timeline, and referral CTA.

### `/guide`

- Uses curated CEX/perp lists from `frontend/lib/exchanges.ts`.
- No backend exchange-directory dependency.
- Pulls referral-link overrides from `/api/home/exchange-ref-links` server-side.
- DB override by slug takes precedence over the curated default.

### `/` and `/trending`

- Homepage content is static and does not call `/api/home/*`.
- Trending is still a simulated "coming soon" reveal, not a live data view.

## Shared UI State

The main shared surfaces:

- navbar and mobile drawer/full-screen sheet
- footer
- language switcher (next-intl)
- theme toggle
- global page transitions and scroll reveal
- refresh buttons + smart polling countdown on data-heavy pages
- data error banners with retry
- scroll-to-top and short-page footer behavior

## API/Environment Behavior

- Server Components use `INTERNAL_API_URL` (default `http://127.0.0.1:1000`).
- Browser fetches use relative `/api/*` unless `NEXT_PUBLIC_API_URL` points to an external host.
- Next.js rewrites `/api/:path*` to the backend.
- Security headers and a report-only CSP are set in `next.config.ts`.
