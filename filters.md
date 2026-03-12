# Filter Configuration

All filtering thresholds are centralized in `backend/src/config/filters.ts`.

## Global Blacklist

Protocols excluded from ALL syncs:
- `francium`
- `growihf`

## Lending Pools

### TVL Thresholds by Asset

| Asset Type | Minimum TVL | Note |
|------------|-------------|------|
| Stablecoins (USDC, USDT, DAI) | $2,000,000 | |
| BTC, WBTC | $10,000,000 | |
| ETH, WETH, WSTETH | $10,000,000 | |
| SOL, USOL | $1,000,000 | |
| HYPE variants | $1,000,000 | Always $1M regardless of chain |
| Hyperliquid chain (non-HYPE) | $500,000 | |
| KiloEx USDC/USDT | $0 | No minimum |
| KiloEx other assets | $500,000 | |

### Minimum APY

| Asset Type | Minimum APY |
|------------|-------------|
| Stablecoins | 1.0% |
| Others (BTC, ETH, SOL, HYPE) | 0.1% |

### Target Symbols

**Stablecoins:**
`USDC`, `USDT`, `DAI`, `USDC.E`, `USDBC`, `USDT.E`, `USDT0`, `USDT.e`, `USDC.e`, `USD₮0`

**ETH variants:**
`ETH`, `STETH`, `WSTETH`, `WETH`

**BTC variants:**
`BTC`, `WBTC`, `CBBTC`

**SOL variants:**
`SOL`, `WSOL`, `UBTC`, `UETH`, `USOL`

**HYPE variants (whitelist):**
`HYPE`, `WHYPE`, `STHYPE`, `KHYPE`, `WSTHYPE`

## Vaults

| Filter | Value |
|--------|-------|
| Minimum TVL | $200,000 |
| Minimum APY | None (show all including negative) |

**Allowed Deposit Tokens:**
`USDC`, `USDT`, `USDT0`, `USDC.E`, `USDBC`, `USDT.E`, `USDT.e`, `USDC.e`, `USD₮0`

## Delta Neutral Strategies

| Filter | Value |
|--------|-------|
| Min Funding Rate | 0.01% (1 basis point) |
| Min Spread | 0.05% |
| Max Slippage | 0.5% |
| Min Liquidity | $100,000 |

## Arbitrage Opportunities

| Filter | Value |
|--------|-------|
| Min Price Difference | 0.1% |
| Min Profit After Fees | 0.05% |
| Max Execution Time | 30 seconds |
| Min Volume (24h) | $50,000 |

Formula: `price_diff = (longPrice - shortPrice) / longPrice * 100`

## Updating Filters

Edit `backend/src/config/filters.ts` and restart the sync service:

```bash
pnpm sync:lending   # Re-sync lending pools
pnpm sync:vault     # Re-sync vaults
```
