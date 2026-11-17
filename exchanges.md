# Exchanges and Protocol Coverage

The backend exchange registry lives in `backend/src/exchanges/index.ts`. `/api/exchanges` is the live source of truth for venue metadata (name, abbreviation, type, native funding period, spot availability, delta-neutral support).

`BLACKLIST_EXCHANGES` defaults to `MEXC,HTX`. Blacklisted venues are removed from ingestion and every API surface.

## Delta-Neutral Sources

Same-exchange spot + short-perp sync uses the curated list in `backend/src/config/deltaExchanges.ts`:

- Hyperliquid
- Lighter
- Aster
- Binance
- OKX
- Bitfinex
- Bybit
- Bitget
- Gate.io
- MEXC
- KuCoin
- HTX

With the default blacklist, 10 of these are active.

### Cross-Exchange Perp-Only Short Legs

`DEFAULT_CROSS_PERP_ONLY_EXCHANGE_NAMES` adds these curated perp-only venues:

- dYdX
- Extended
- EdgeX
- Paradex
- ApeX
- Avantis
- GMX
- Nado
- Ostium
- Reya
- Pacifica
- Variational
- RiseX
- Grvt
- Aftermath
- Trade.xyz

Override with `DELTA_CROSS_PERP_EXCHANGES` (comma-separated). Set it to an empty value to disable all perp-only venues.

## Arbitrage Scanner Sources

The registry contains **54 venues**; with the default blacklist **52 are active**. `hasSpot=true` marks venues that expose spot markets for delta-neutral metadata.

**Established venues (29):**

- Hyperliquid
- Lighter
- Extended
- Pacifica
- Reya
- Variational
- Aster
- Grvt
- Paradex
- EdgeX
- Binance
- OKX
- Bitfinex
- Bybit
- Bitget
- Gate.io
- MEXC
- KuCoin
- HTX
- Ostium
- RiseX
- dYdX
- BingX
- Backpack
- GMX
- Avantis
- Nado
- ApeX
- Aftermath

**Emerging / prediction venues (25):**

- Gains Network
- StandX
- Trade.xyz
- Decibel
- Hibachi
- Katana
- TxFlow
- 01 Exchange
- Vest Exchange
- Arcus
- Bulk
- Bullet
- Entropy
- Hello Trade
- Hotstuff
- Kalshi
- KM
- Lighter RH
- Ondo Finance
- Paper Trade
- Perpl
- Phoenix
- Polymarket
- Popdex
- QFEX

Spot-enabled venues: Binance, OKX, Hyperliquid, Lighter, Aster, Bitfinex, Bybit, Bitget, Gate.io, MEXC, KuCoin, HTX, BingX, Backpack. MEXC and HTX are blacklisted by default, so they do not appear in `/api/exchanges`.

## Vault Fetchers

Registered in `backend/src/services/VaultSyncService.ts` (14 fetchers):

- Hyperliquid
- Jupiter
- Paradex
- Extended
- Lighter
- GRVT
- Reya
- Ostium
- Bluefin
- KiloEx
- DefiLlama
- RiseX
- ApeX
- EdgeX

DefiLlama also supplies general vault data and APR history for `DL-*` vault IDs.

## RWA Registry

`rwa_assets` is a shared, symbol-keyed registry used to classify tokenized stocks, commodities, indices and FX across venues.

- Seeded by `pnpm migrate:rwa-assets` from `backend/src/config/rwaSymbols.ts`
- Learned from venue metadata by `pnpm rwa:sync`
- Curated with `pnpm rwa:add` / `pnpm rwa:remove`
- Runtime cache TTL: 5 minutes

## Curated Guide Exchange List

The guide page does not depend on exchange cron data. It uses the curated frontend source of truth in `frontend/lib/exchanges.ts`, ordered by `rank`.

### CEX order (11)

1. Binance
2. OKX
3. Bybit
4. Coinbase
5. KuCoin
6. Gate
7. Bitget
8. MEXC
9. HTX
10. Bitfinex
11. Kraken

### Perp order (13)

1. Hyperliquid
2. Aster
3. Lighter
4. edgeX
5. Variational
6. GMX
7. Drift Trade
8. Extended
9. Derive
10. Grvt
11. Nado
12. Avantis
13. RiseX

Referral links on `/guide` are resolved as:

1. `ref_links` override from DB by slug
2. curated default link

## Notes

- Exchange metadata sync is not a cron job; `/api/exchanges` is derived from the live registry and cached for 12 hours.
- Homepage exchange count is still served by `/api/home/total-exchanges`, but the homepage renders a static `380+` stat.
- Guide visuals, copy and ordering should be updated in frontend curated data, not through backend cron jobs.
