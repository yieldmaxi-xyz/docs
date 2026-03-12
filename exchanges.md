# Integrated Exchanges

CryptoYieldMaxi aggregates data from 22+ exchanges.

## Centralized Exchanges (CEX)

| Exchange | Funding Interval | Features |
|----------|------------------|----------|
| Binance | 8h | Largest volume |
| Bybit | 8h | Popular perps |
| OKX | 8h | Deep liquidity |
| Bitget | 8h | Copy trading |
| Gate.io | 8h | Wide selection |
| MEXC | 8h | New listings |
| KuCoin | 8h | Established |
| HTX | 8h | Asian market |
| Bitfinex | 8h | Pro traders |

## Decentralized Exchanges (DEX)

### Perp DEXs

| Exchange | Chain | Funding Interval | Notes |
|----------|-------|------------------|-------|
| **Hyperliquid** | Hyperliquid L1 | 1h | Top DEX by volume |
| **Lighter** | Arbitrum | 1h | Order book DEX |
| **Aster** | Arbitrum | 8h | Perpetual DEX |
| **Variational** | Arbitrum | 8h | AMM-based |
| **Extended** | Extended L2 | 1h | Leveraged trading |
| **GRVT** | zkSync | 8h | Hybrid DEX |
| **Ostium** | Arbitrum | 8h | RWA perps |
| **Paradex** | StarkEx | 8h | StarkWare based |
| **EdgeX** | EdgeL2 | 1h | Low fees |
| **Pacifica** | Solana | 8h | Solana perps |
| **Reya** | Arbitrum | 8h | Modular DEX |

## Funding Rate Intervals

### 1-Hour Funding
- Hyperliquid
- Lighter
- Extended
- EdgeX

### 8-Hour Funding
- All CEXs (Binance, Bybit, etc.)
- Variational (4 periods × 8h = normalized to hourly)
- Aster, GRVT, Ostium, Paradex

## Exchange Features

### Vaults Available
| Exchange | Vault Type |
|----------|-----------|
| Hyperliquid | HLP (Liquidity Provider) |
| Jupiter | JLP |
| GRVT | Market Making |
| Ostium | USDT Vault |
| Bluefin | UDL Vaults |
| KiloEx | USDC/USDT Vaults |

### Lending Integration
| Protocol | Source |
|----------|--------|
| Aave V3 | DefiLlama |
| Compound | DefiLlama |
| Pendle | DefiLlama |
| Hyperlend | DefiLlama |
| Venus | DefiLlama |
| Kamino | DefiLlama |

## Adding New Exchange

1. Create client in `backend/src/exchanges/newexchange.ts`
2. Implement `ExchangeClient` interface:
   ```typescript
   interface ExchangeClient {
     name: string;
     type: 'cex' | 'perp_dex' | 'spot_dex';
     getFundingRates(symbols: string[]): Promise<FundingSnapshot[]>;
   }
   ```
3. Register in `backend/src/exchanges/index.ts`
4. Add to sync services as needed

## API Base URLs

| Exchange | API Base URL |
|----------|--------------|
| Hyperliquid | https://api.hyperliquid.xyz |
| Lighter | https://api.lighter.xyz |
| Binance | https://fapi.binance.com |
| OKX | https://www.okx.com |
| Variational | https://api.variational.io |
