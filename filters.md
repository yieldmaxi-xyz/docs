# Filter Configuration

There are two layers of filtering:

1. **Centralized sync filters** in `backend/src/config/filters.ts` (used by lending and vault syncs).
2. **Runtime/environment filters** in `backend/src/config/env.ts` and the sync/query engines (used by delta-neutral and arbitrage).

> The `FilterConfig.deltaNeutral` and `FilterConfig.arbitrage` blocks still exist for reference, but current delta-neutral and arbitrage runtime thresholds come from the environment values and the query/engine logic described below.

## Global Protocol Blacklist

Protocols excluded from ALL syncs (`FilterConfig.global.blacklist`):

- `francium`
- `growihf`

## Exchange Blacklist

Controlled by `BLACKLIST_EXCHANGES`.

- Default: `MEXC,HTX`
- Set it to an empty string to explicitly disable aliases/blacklisting.
- Matching is normalized and understands common aliases (`mxc`, `huobi`, `huobipro`, `gateio`, `gate.io`, `binancecex`).
- Blacklisted venues are excluded from ingestion, delta-neutral outputs, the arbitrage scanner and `/api/exchanges`.

## Lending Pools

### TVL Thresholds by Asset

| Asset Type | Minimum TVL | Note |
| --- | --- | --- |
| Stablecoins (USDC, USDT, DAI) | $2,000,000 | |
| BTC / WBTC | $10,000,000 | |
| ETH / WETH / stETH / wstETH | $10,000,000 | |
| SOL / uSOL / wSOL | $1,000,000 | |
| HYPE variants | $1,000,000 | Always $1M regardless of chain |
| Hyperliquid chain (non-HYPE) | $500,000 | |
| KiloEx USDC/USDT | $0 | No minimum |
| KiloEx other assets | $500,000 | |

### Minimum APY

| Asset Type | Minimum APY |
| --- | --- |
| Stablecoins | 1.0% |
| Others (BTC, ETH, SOL, HYPE) | 0.1% |

### Symbol Groups

**Stablecoins:**
`USDC`, `USDT`, `DAI`, `USDC.E`, `USDBC`, `USDT.E`, `USDT0`, `USDT.e`, `USDC.e`, `USD₮0`

**ETH variants:**
`ETH`, `STETH`, `WSTETH`, `UETH`, `WETH`, `RETH`, `CBETH`, `FRXETH`, `SETH2`

**BTC variants:**
`BTC`, `WBTC`, `BTCB`, `CBBTC`, `BTC.B`, `UBTC`

**SOL variants:**
`SOL`, `USOL`, `WSOL`, `MSOL`, `JITOSOL`, `BSOL`, `STSOL`, `UBTC`, `UETH`

**HYPE variants (whitelist):**
`HYPE`, `WHYPE`, `STHYPE`, `KHYPE`, `WSTHYPE`

The API additionally returns up to 50 pools per category ordered by APY and excludes Pendle PT-HYPE pools from the HYPE section.

## Vaults

| Filter | Value |
| --- | --- |
| Minimum TVL | $200,000 |
| Minimum APY | None (shows negative APY too) |

**Allowed deposit tokens:**
`USDC`, `USDT`, `USDT0`, `USDC.E`, `USDBC`, `USDT.E`, `USDT.e`, `USDC.e`, `USD₮0`

## Delta-Neutral Strategies

Runtime values from `backend/src/config/env.ts` (overridable via `.env`):

| Filter | Env var | Default |
| --- | --- | --- |
| Min spot volume | `DELTA_MIN_SPOT_VOLUME` | `$100,000` |
| Min perp volume | `DELTA_MIN_PERP_VOLUME` | `$200,000` (`.env.example` uses `500000`) |
| Min perp open interest | `DELTA_MIN_PERP_OI` | `$300,000` (`.env.example` uses `1000000`) |
| Max cross-exchange basis | `DELTA_CROSS_MAX_BASIS_PCT` | `3%` |
| Cross perp-only venues | `DELTA_CROSS_PERP_EXCHANGES` | curated list in `deltaExchanges.ts` |
| Cross route cap | `DELTA_CROSS_TOP_ROUTES_PER_SYMBOL` | `0` (store all) |

Additional quality rules:

- Funding rates are capped at **10% per 8h** (`MAX_REASONABLE_RATE = 0.1`) before APR calculation.
- Aggregated periods require at least **70% window coverage** and a fresh newest sample (3h for 24h, 6h for 7d, 24h for 30d, 48h for 90d/180d).
- Commodity/non-spot symbols in `DELTA_EXCLUSIONS` are skipped (precious metals, energy contracts, legacy commodity names).
- Tokenized stocks/RWA are **not** excluded and trade 24/7; the old weekend filter is a no-op.

## Arbitrage Scanner

Scanner pair construction does not apply a hard default liquidity floor. The UI and API filters do:

| Filter | Default in UI/API |
| --- | --- |
| Pair min volume (`min_vol`) | Not applied unless supplied |
| Pair min OI (`min_oi`) | Not applied unless supplied |
| Net APR (`min_apr` / `max_apr`) | Not applied unless supplied |
| Basis spread (`min_in_spread` / `max_in_spread`) | Not applied unless supplied |

`ARBITRAGE_MIN_PERP_VOLUME` (default `$200,000`) and `ARBITRAGE_MIN_PERP_OI` (default `$100,000`) are used by reporting helpers, not as a hard scanner ingest cutoff.

Mode filtering is mandatory when a mode is supplied:

- `mode=funding` → `funding_spread_apr > 0`
- `mode=basis` → `basis_pct > 0`
- `mode=realized` → `funding_arb_24h_pct > 0` OR `funding_arb_7d_pct > 0`

## RWA / Asset Class

The shared `rwa_assets` registry is the authoritative symbol classification used by the arbitrage scanner.

- Categories: `stock`, `commodity`, `index`, `forex`, `premarket`, `rwa`
- Venue metadata is used to learn new symbols.
- An explicit `CRYPTO` classification on every reporting leg wins over a registry match, preventing stock/crypto ticker collisions.
- Manage entries with:

```bash
pnpm rwa:list
pnpm rwa:add -- NVDA stock
pnpm rwa:remove -- NVDA
pnpm rwa:sync     # learn from venue metadata
pnpm rwa:audit    # dry-run learning
```

## Updating Filters

Edit the relevant source and restart/re-sync:

```bash
# Centralized lending/vault thresholds
backend/src/config/filters.ts

# Environment overrides
backend/.env
```

Manual re-sync:

```bash
pnpm sync:lending
pnpm sync:vault
pnpm sync:delta
pnpm sync:arbitrage
```
