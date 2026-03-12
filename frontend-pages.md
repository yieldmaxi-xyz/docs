# Frontend Pages

## Routes Overview

| Route | Page | Description |
|-------|------|-------------|
| `/` | Home | Landing page with teasers |
| `/lending` | Lending | DeFi lending pools |
| `/vaults` | Vaults | Delta-neutral vaults |
| `/arbitrage` | Arbitrage | Funding rate arbitrage |
| `/delta-neutral` | Delta Neutral | Funding extraction |
| `/trending` | Trending | Top opportunities |
| `/airdrop` | Airdrop | Airdrop programs |
| `/guide` | Guide | Exchange guides |
| `/about` | About | Project info |

## Page Details

### Home Page (`/`)

**Features:**
- Hero section with animated stats
- Top opportunities teasers (Lending, Vaults, Arbitrage)
- Featured exchanges (CEX + Perp DEX)
- Educational content sections
- Quick links to all features

**Data Source:** `/api/home/teasers`

---

### Lending Page (`/lending`)

**Features:**
- Token tabs: USDC, USDT, ETH, BTC, SOL, HYPE
- Summary stats: Opportunities, Top APR, Avg APR, Best Protocol
- Sortable table with columns:
  - Rank, Protocol, Chain, Token, TVL, APR, Base, Reward, 30D, Action
- Search and filter by chain/protocol

**Data Source:** `/api/lending?token=USDC`

---

### Vaults Page (`/vaults`)

**Features:**
- Grid/table view toggle
- Vault cards with:
  - Protocol name, Chain, Deposit token
  - TVL, APY, Strategy type
  - Notice/warning popup
- Filter by chain

**Data Source:** `/api/vaults`

---

### Arbitrage Page (`/arbitrage`)

**Features:**
- Period toggle: 1h / 8h
- Exchange filter
- Table with:
  - Symbol, Long Exchange, Short Exchange
  - Long Rate, Short Rate, Edge
  - APR, Volume, Action

**Data Source:** `/api/arbitrage?period=1h`

---

### Delta Neutral Page (`/delta-neutral`)

**Features:**
- Funding extraction opportunities
- Table with:
  - Symbol, Exchange, Funding Rate
  - APR, Time Cost, Spread/Fee
- Strategy explanation cards

**Data Source:** `/api/delta-neutral`

---

### Airdrop Page (`/airdrop`)

**Features:**
- Card-based layout
- Each card shows:
  - Exchange logo, Name
  - Description, How to Earn
  - Estimated Value badge
  - Timeline, CTA button

**Data Source:** `/api/airdrop`

---

### Guide Page (`/guide`)

**Features:**
- Exchange listing with guides
- Registration links with referral codes
- Chain/network information

**Data Source:** `/api/exchanges`

---

### About Page (`/about`)

**Features:**
- Project introduction
- Author bio
- Donation addresses (BTC, EVM, SOL)
- Social links

**Data Source:** Static content

## Components Structure

```
frontend/components/
├── ui/                 # Base UI components
│   ├── Button.tsx
│   ├── Card.tsx
│   ├── Table.tsx
│   └── Badge.tsx
├── layout/             # Layout components
│   ├── Header.tsx
│   ├── Footer.tsx
│   └── Navigation.tsx
├── lending/            # Lending-specific
│   └── LendingTable.tsx
├── vaults/             # Vault-specific
│   └── VaultCard.tsx
└── shared/             # Shared components
    ├── ExchangeLogo.tsx
    ├── ChainBadge.tsx
    └── APRDisplay.tsx
```

## Styling

- **Framework:** Tailwind CSS
- **Theme:** Dark mode default
- **Colors:**
  - Green: Positive yields
  - Red: Negative/risk
  - Blue: Links/actions
  - Purple: Highlights

## Data Fetching

- Server Components for initial load
- Client Components for filtering/sorting
- SWR for real-time updates (optional)
