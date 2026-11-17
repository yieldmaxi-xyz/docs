# Exchange Symbol Mapping Reference

**Last Updated:** 2026-11-17

This document describes how venue-native symbols are converted into comparable assets for delta-neutral and arbitrage calculations.

The implementation lives in:

- `backend/src/utils/symbolNormalizer.ts`
- `backend/src/services/DeltaSyncService.ts`
- `backend/src/services/CrossDeltaService.ts`
- `backend/src/services/ArbitrageScannerEngine.ts`
- `backend/src/services/SharedAssetClass.ts`

## Matching Pipeline

```text
venue-native symbol
  -> client.normalizeSymbol()
  -> normalizeCanonicalAssetSymbol()
  -> stripPerpMultiplier() for perp grouping
  -> canonical asset key used for spot/perp or long/short pairing
```

Key helpers:

| Helper | Purpose |
| --- | --- |
| `normalizeSpotSymbol` | Strips U/H/M/O wrappers and trailing numbers on Hyperliquid-style spot symbols |
| `normalizePerpSymbol` | Uppercases, removes `-USD` suffix and delimiters |
| `normalizeKPrefix` | Maps meme assets to their 1000-prefix perp form |
| `stripKPrefix` | `1000PEPE` → `PEPE` |
| `parsePerpSymbolMultiplier` | Reads contract multipliers: `1000`, `10000`, `100000`, `1000000` |
| `stripPerpMultiplier` | Removes the multiplier prefix for grouping |
| `smartMatch` | Fallback spot/perp matcher for wrapped and prefixed symbols |
| `normalizeCanonicalAssetSymbol` | Final cross-venue canonical ticker, alias and wrapper resolver |

## Canonical Ticker Rules

`normalizeCanonicalAssetSymbol` handles:

- namespace prefixes (`xyz:BRENTOIL` → `BRENTOIL`)
- quote suffixes (`BTC/USDT`, `BTC-USDC`, `BTC-USD.P`, `BTC-PERP`, `BTC-PERP_USDT0`)
- Extended 24/5 suffix (`AAPL_24_5-USD` → `AAPL`)
- Gains equity suffix (`TSLA_1` → `TSLA`)
- Backpack equity suffix (`NVDA.US` → `NVDA`)
- Kalshi wrappers (`KXNVDAPERP` → `NVDA`)
- stock wrappers such as MEXC/Bitget/Bybit `NVDASTOCK` → `NVDA`
- canonical aliases (for example `GOLD`/`GC` → `XAU`, `SILVER` → `XAG`, `WTI`/`CL` → `WTIOIL`, `US100`/`NDX` → `NAS100`, `SPX500` → `SP500`)

The function is memoized (bounded cache) because it is called millions of times in the scanner pairing loop.

## Contract Multipliers

Exchanges quote perp prices per contract unit. The multiplier is parsed so basis comparisons use the token price:

| Perp symbol | Multiplier | Base asset |
| --- | --- | --- |
| `PEPE` | `1` | PEPE |
| `1000PEPE` | `1000` | PEPE |
| `10000CAT` | `10000` | CAT |
| `1000000MOG` | `1000000` | MOG |

Delta-neutral divides the perp price by the multiplier before comparing it with spot. The scanner groups multiplier variants under one base ticker so contracts like `PEPE` and `1000PEPE` can pair.

## K-Prefix Symbols

`normalizeKPrefix` currently maps:

- PEPE → 1000PEPE
- BONK → 1000BONK
- SHIB → 1000SHIB
- FLOKI → 1000FLOKI
- LUNC → 1000LUNC
- XEC → 1000XEC
- AIDOGE → 1000AIDOGE

Venue behavior:

- Binance / Bybit: already use 1000-prefix perps
- MEXC / KuCoin / Bitget / Gate.io / HTX: apply `normalizeKPrefix`
- OKX / Bitfinex: map to 1000-prefix where needed
- Hyperliquid: U-prefixed wrapped spot symbols are stripped by `normalizeSpotSymbol`

## Delta-Neutral Exclusions

`DELTA_EXCLUSIONS` in `DeltaSyncService.ts` currently excludes:

**Precious metals:** `XAG`, `XAU`, `XPT`, `XPD`

**Energy commodities:** `CL`, `NG`, `HO`, `RB`

**Legacy names:** `SILVER`, `SLIVER`, `GOLD`, `WTI`, `BRENT`

Tokenized gold (`XAUT`, `PAXG`) is intentionally **not** excluded. Tokenized stocks/RWA are not excluded either; the old weekend filter is a no-op because these markets trade 24/7.

## Funding Periods

| Venue | Native period | Normalization |
| --- | --- | --- |
| Binance | 8h | Direct |
| OKX | 8h | Direct |
| Bybit | 8h | Direct |
| Bitget | 8h | Direct |
| Gate.io | 8h | Direct |
| KuCoin | 8h | Direct |
| HTX | 8h | Direct |
| Bitfinex | 8h | Direct |
| Lighter | 8h | Direct |
| Aster | 8h | Direct |
| Hyperliquid | 1h | ×8 |
| Ostium | 1h | Per-second × 3600 × 8 |
| MEXC | Variable | Check contract details |
| GRVT | Variable | Check contract details |
| Paradex | Variable | Check contract details |
| Reya | Variable | Check contract details |
| Variational | Variable | Check contract details |
| EdgeX | Variable | Check contract details |
| RiseX | Variable | Check contract details |

Formula:

```text
rate8h = nativeRate × (8 / nativePeriodHours)
```

## Volume and Liquidity Filtering

Exchange clients can implement `getSpotSymbolsWithVolume()`. When present, zero-volume spot symbols are dropped before matching:

```ts
if (client.getSpotSymbolsWithVolume) {
  const spotVolumeMap = await client.getSpotSymbolsWithVolume();
  spotSymbols = [...spotVolumeMap.keys()].filter((s) => (spotVolumeMap.get(s) || 0) > 0);
}
```

Volume/Open-Interest floors for delta-neutral come from:

- `DELTA_MIN_SPOT_VOLUME` (default 100,000)
- `DELTA_MIN_PERP_VOLUME` (default 200,000; `.env.example` uses 500,000)
- `DELTA_MIN_PERP_OI` (default 300,000; `.env.example` uses 1,000,000)

## Funding Rate Validation

`DeltaSyncService` caps extreme rates at **10% per 8h** (`MAX_REASONABLE_RATE = 0.1`). Values above the cap are clamped to the cap with the original sign, and a warning is logged.

Typical ranges:

- Normal: 0.0001–0.01 (0.01%–1% per 8h)
- High: 0.01–0.05 (1%–5% per 8h)
- Extreme: >0.1 (>10% per 8h) → capped

## Asset Class / RWA Wrappers

RWA detection recognizes venue wrappers even before canonicalization:

- `*STOCK` suffix (MEXC / Bitget / Bybit)
- `NCSK*`, `NCCO*`, `NCSI*`, `NCFX*` (BingX)
- `*_24_5*` (Extended)
- `*.US` (Backpack)
- `KX*PERP` (Kalshi)

The shared `rwa_assets` registry is authoritative at runtime. See [filters.md](filters.md#rwa--asset-class) for management commands.

## Troubleshooting

**Symbol missing from delta-neutral**

1. Check `DELTA_EXCLUSIONS` (commodities).
2. Check that both spot and perp markets exist on the same venue.
3. Check spot volume > 0.
4. Check that spot and perp canonicalize to the same key.
5. Check the funding-rate cap and venue-specific interval conversion.

**Extreme APR values**

1. Inspect the venue's native funding format (decimal vs percentage).
2. Verify the interval conversion (`rate8h = nativeRate × (8 / nativePeriod)`).
3. Confirm the 10% per-8h cap is applied.
4. Check spot/perp price scaling when a contract multiplier is involved.

**Mismatched spot/perp or long/short pairs**

1. Inspect the venue `normalizeSymbol()` output.
2. Verify 1000/10000/1000000 multiplier handling.
3. Check canonical aliases and RWA wrappers.
4. Confirm both legs reach the same `normalizeCanonicalAssetSymbol()` result.
