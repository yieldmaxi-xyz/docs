# System Architecture

## Overview

CryptoYieldMaxi is a monorepo with two main components:
- **Backend**: Express API + Cron scheduler
- **Frontend**: Next.js App Router

## Data Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           EXTERNAL APIS                                  │
│                                                                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │Hyperliquid│ │  Lighter │ │  Binance │ │   OKX    │ │DefiLlama │      │
│  └─────┬─────┘ └─────┬────┘ └─────┬────┘ └─────┬────┘ └─────┬────┘      │
└────────┼─────────────┼───────────┼───────────┼───────────┼──────────────┘
         │             │           │           │           │
         ▼             ▼           ▼           ▼           ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         CRON SCHEDULER                                   │
│                                                                          │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐             │
│  │ Arbitrage Sync │  │  Vault Sync    │  │ Lending Sync   │             │
│  │   (Hourly: 03) │  │   (Hourly: 09) │  │   (Hourly: 07) │             │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘             │
└──────────┼───────────────────┼───────────────────┼──────────────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         SYNC SERVICES                                    │
│                                                                          │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐             │
│  │ ArbitrageSvc   │  │   VaultSvc     │  │  LendingSvc    │             │
│  │ DeltaNeutralSvc│  │                │  │                │             │
│  │                │  │                │  │                │             │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘             │
└──────────┼───────────────────┼───────────────────┼──────────────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                            DATABASE                                      │
│                                                                          │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │
│  │ arbitrage_  │ │   vaults    │ │  lending_   │ │  airdrops   │        │
│  │opportunities│ │             │ │   pools     │ │             │        │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘        │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                        │
│  │   funding_  │ │ exchanges   │ │ ref_links   │                        │
│  │  extractors │ │             │ │             │                        │
│  └─────────────┘ └─────────────┘ └─────────────┘                        │
└─────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          API ROUTES                                      │
│                                                                          │
│  GET /api/lending      GET /api/vaults       GET /api/delta-neutral     │
│  GET /api/arbitrage    GET /api/airdrop      GET /api/exchanges         │
└─────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          FRONTEND                                        │
│                                                                          │
│  /lending    /vaults    /delta-neutral    /arbitrage    /airdrop        │
└─────────────────────────────────────────────────────────────────────────┘
```

## Cron Schedule

| Job | Interval | Description |
|-----|----------|-------------|
| Delta-Neutral Sync | Every hour (:01) | Calculates single-exchange spot-perp funding |
| Arbitrage Sync | Every hour (:03) | Multi-exchange perp-perp spread arbitrage |
| Lending Sync | Every hour (:07) | Lending pools from DefiLlama |
| Vault Sync | Every hour (:09) | Delta-neutral vaults from custom fetchers |
| Telegram Summary | Every hour (:15) | Dispatches RAW health monitoring stats |
| Exchange Meta Sync | Daily (01:00) | Pulls exchange TVL/metadata from DefiLlama |

## Database Models

| Model | Table | Description |
|-------|-------|-------------|
| LendingPool | lending_pools | DeFi lending opportunities |
| Vault | vaults | Delta-neutral vaults |
| ArbitrageOpportunity | arbitrage_opportunities | Funding arbitrage |
| FundingExtractor | funding_extractors | Delta-neutral strategies |
| Exchange | exchanges | CEX and DEX metadata |
| Airdrop | airdrops | Airdrop programs |
| RefLink | ref_links | Referral links by protocol |

## Backend Folder Structure

```
backend/src/
├── app.ts              # Express app setup
├── server.ts           # HTTP server
├── worker.ts           # Cron worker entry
├── config/
│   ├── database.ts     # Sequelize config
│   └── filters.ts      # TVL/APY thresholds
├── exchanges/          # 22 exchange clients
│   ├── hyperliquid.ts
│   ├── lighter.ts
│   ├── binance.ts
│   └── ...
├── models/             # Sequelize models
├── routes/             # API endpoints
├── services/           # Business logic
├── cron/               # Scheduled jobs
├── scripts/            # Manual sync scripts
└── utils/              # Helpers (http, logger)
```

## Frontend Folder Structure

```
frontend/app/
├── page.tsx            # Home page
├── layout.tsx          # Root layout
├── lending/page.tsx
├── vaults/page.tsx
├── arbitrage/page.tsx
├── delta-neutral/page.tsx
├── trending/page.tsx
├── airdrop/page.tsx
├── guide/page.tsx
└── about/page.tsx
```
