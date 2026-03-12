# CryptoYieldMaxi

A comprehensive crypto yield aggregation platform that helps users discover:
- **Lending opportunities** across DeFi protocols
- **Delta-neutral vaults** for low-risk yield
- **Funding arbitrage** between exchanges
- **Airdrop opportunities** from exchanges

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         FRONTEND                                 │
│  Next.js App Router + Tailwind CSS                              │
│  Pages: Home, Lending, Vaults, Arbitrage, Airdrop               │
└──────────────────────────┬──────────────────────────────────────┘
                           │ REST API
┌──────────────────────────▼──────────────────────────────────────┐
│                         BACKEND                                  │
│  Express + TypeScript + Sequelize                               │
│                                                                  │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐                │
│  │   Cron     │  │  Services  │  │   Routes   │                │
│  │  Scheduler │→ │  (Sync)    │→ │   (API)    │                │
│  └────────────┘  └────────────┘  └────────────┘                │
│         │                                                        │
│  ┌──────▼───────────────────────────────────────┐               │
│  │              Exchange Clients                 │               │
│  │  Hyperliquid, Lighter, Binance, OKX, ...     │               │
│  └───────────────────────────────────────────────┘               │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                         DATABASE                                 │
│  MySQL + Sequelize ORM                                          │
│  Tables: lending_pools, vaults, arbitrage_opportunities, etc.   │
└─────────────────────────────────────────────────────────────────┘
```

## Quick Start

### Prerequisites
- Node.js 20+
- MySQL 8+
- pnpm

### Backend Setup
```bash
cd backend
pnpm install
cp .env.example .env  # Configure database
pnpm dev:api          # Start API server (port 1000)
pnpm dev:cron         # Start cron scheduler
```

### Frontend Setup
```bash
cd frontend
pnpm install
pnpm dev              # Start Next.js (port 1001)
```

## Environment Variables

### Backend
```env
NODE_ENV=development
PORT=1000
DB_HOST=localhost
DB_PORT=3306
DB_NAME=crypto_yield_maxi
DB_USER=root
DB_PASSWORD=
```

### Frontend
```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:1000
```

## Documentation

- [Architecture](./architecture.md) - System design and data flow
- [Filters](./filters.md) - TVL/APY thresholds and blacklists
- [Exchanges](./exchanges.md) - Integrated exchanges (22+)
- [API Reference](./api.md) - REST API endpoints
- [Frontend Pages](./frontend-pages.md) - Page routes and features
- [Deployment](./deployment.md) - PM2, cluster mode, production setup

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Next.js 15, React 19, Tailwind CSS, shadcn/ui |
| Backend | Express, TypeScript, Sequelize, Helmet |
| Database | MySQL 8, Redis |
| Cron | node-cron, in-memory sync reporting |
| HTTP | Axios with retry |

## Project Structure

```
/
├── backend/
│   └── src/
│       ├── config/      # Database, filters
│       ├── exchanges/   # 22 exchange clients
│       ├── models/      # Sequelize models
│       ├── routes/      # API endpoints
│       ├── services/    # Business logic
│       ├── cron/        # Scheduled jobs
│       └── scripts/     # Manual sync scripts
├── frontend/
│   └── app/
│   │   ├── lending/     # Lending pools page
│   │   ├── vaults/      # Delta-neutral vaults
│   │   ├── arbitrage/   # Funding arbitrage
│   │   └── airdrop/     # Airdrop opportunities
└── docs/                # This documentation
```

## License

MIT
