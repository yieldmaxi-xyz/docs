# API Reference

Base URL: `http://localhost:1000` (development) or `https://api.yourdomain.com` (production)

## Endpoints

### GET /api/lending

Returns lending pool opportunities grouped by token.

**Query Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| token | string | Filter by token (USDC, USDT, ETH, BTC, SOL, HYPE) |

**Response:**
```json
{
  "data": {
    "USDC": [
      {
        "pool_id": "uuid",
        "project": "Aave V3",
        "chain": "Ethereum",
        "symbol": "USDC",
        "tvl_usd": "50000000.00",
        "apy": "5.25",
        "apy_base": "3.50",
        "apy_reward": "1.75",
        "apy_30d": "5.10",
        "project_url": "https://app.aave.com",
        "logo_url": "/assets/icons/protocols/aave-v3.png",
        "last_updated": "2026-02-08T08:00:00Z"
      }
    ]
  },
  "lastUpdated": "2026-02-08T08:00:00Z"
}
```

---

### GET /api/vaults

Returns delta-neutral vault opportunities.

**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "protocol": "Hyperliquid",
      "name": "HLP",
      "chain": "Hyperliquid",
      "deposit_token": "USDC",
      "tvl_usd": "150000000.00",
      "apy": "25.50",
      "strategy": "Market Making",
      "ref_link": "https://app.hyperliquid.xyz",
      "logo_url": "/assets/icons/protocols/hyperliquid.png"
    }
  ],
  "lastUpdated": "2026-02-08T08:00:00Z"
}
```

---

### GET /api/arbitrage

Returns funding rate arbitrage opportunities between exchanges.

**Query Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| period | string | 1h | Time period: `1h` or `8h` |

**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "symbol": "ETH",
      "long_exchange": "Hyperliquid",
      "short_exchange": "Binance",
      "long_rate_pct": -0.005,
      "short_rate_pct": 0.015,
      "edge_pct": 0.02,
      "apr_pct": 175.2,
      "long_vol_usd": "50000000",
      "short_vol_usd": "500000000"
    }
  ],
  "lastUpdated": "2026-02-08T08:00:00Z"
}
```

---

### GET /api/delta-neutral

Returns delta-neutral funding extraction strategies.

**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "symbol": "BTC",
      "exchange": "Hyperliquid",
      "rate_pct": 0.02,
      "apr_pct": 175.2,
      "time_cost_hours": 24,
      "spread_fee_pct": 0.1,
      "ref_link": "https://app.hyperliquid.xyz"
    }
  ]
}
```

---

### GET /api/airdrop

Returns airdrop farming opportunities.

**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "exchange": "Hyperliquid",
      "name": "Points Season 2",
      "description": "Trade to earn HYPE points",
      "how_to_earn": "Trade perpetuals during the season",
      "estimated_value": "High Opportunity",
      "timeline": "Ongoing",
      "ref_link": "https://app.hyperliquid.xyz",
      "logo_url": "/assets/icons/exchanges/hyperliquid.png"
    }
  ]
}
```

---

### GET /api/exchanges

Returns exchange metadata.

**Query Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| type | string | Filter by type: `cex` or `perp_dex` |

**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "name": "Hyperliquid",
      "type": "perp_dex",
      "chain": "Hyperliquid",
      "volume_24h": "5000000000",
      "open_interest": "2000000000",
      "ref_link": "https://app.hyperliquid.xyz",
      "logo_url": "/assets/icons/exchanges/hyperliquid.png"
    }
  ]
}
```

---

### GET /api/health

Returns system health status.

**Response:**
```json
{
  "status": "ok",
  "database": "connected",
  "uptime": 86400
}
```

---

### GET /api/home/teasers

Returns homepage teaser data.

**Response:**
```json
{
  "topLending": [...],
  "topVaults": [...],
  "topArbitrage": [...]
}
```
