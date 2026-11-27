# Reading the Yield Maxi dashboard

Start with a question about a market. The tables help you find a pair, the calculator lets you test a holding period, and the charts show how that pair behaved over time. Figures in the screenshots reflect the dashboard at the time of capture.

[Arbitrage](#arbitrage) · [Spread Tracking](#spread-tracking) · [Delta Neutral](#delta-neutral) · [Stats](#stats) · [Lending](#lending) · [Vaults](#vaults) · [Airdrop](#airdrop) · [Chart](#chart)

## Arbitrage

![Arbitrage table comparing long and short perpetual contracts across venues](images/arbitrage.png)

**Price** mode is a starting point for comparing entry prices. **Funding** mode is a starting point for comparing recurring funding. Choose **Columns → All columns** to display the full table.

| Column | What it tells you |
| --- | --- |
| **Rank** | The row's position under the current sort. Use the star to add the pair to your watchlist. |
| **Asset** | The underlying asset shared by the two perpetual contracts. |
| **Pair (Long / Short)** | The exchange for the long leg is shown first, followed by the exchange for the short leg. |
| **Prices** | The displayed ask for entering the long and bid for entering the short. |
| **Price Arb** | The entry price difference, shown as a percentage and as a price difference per unit. A positive value means the short bid is above the long ask. |
| **Funding** | The current funding rate for each leg. Check each contract's funding interval when comparing rates. |
| **Funding Arb 1H** | The current short-leg funding rate minus the long-leg rate, expressed per hour. An **≈** marks a conversion from a different funding interval. |
| **Funding Arb 24H** | The cumulative difference in settled funding for the two legs over the past day, when both histories support it. |
| **Funding Arb 7D** | The cumulative difference in settled funding over the past seven days, when both histories support it. |
| **Est. Fees** | Estimated trading fees for opening and closing both legs at the table's reference position size. |
| **Net APR 1H** | Estimated annualized return using current funding conditions, entry prices, and modeled costs. The calculator next to it lets you change the position size and holding period. |
| **Net APR 24H** | Estimated annualized net return using the 24H funding history. |
| **Net APR 7D** | Estimated annualized net return using the 7D funding history. |
| **Volume (24H)** | The lower 24-hour trading volume across the two contracts. |
| **Open Interest** | The lower open interest across the two contracts. |
| **Break-Even** | Estimated hours of funding income required to cover the modeled round-trip fees. |
| **Action** | **Tracking** opens the historical charts for the exact long and short contracts in the row. |

### Follow a funding opportunity

Open **Arbitrage** and choose **Funding**. Search for an asset and read each row from left to right. The first exchange is the long leg and the second is the short leg. Check the price gap and estimated fees before reading Net APR. Select **Tracking** to open the same two contracts in Spread Tracking. [Open Arbitrage](https://yieldmaxi.xyz/arbitrage)

Suppose a pair shows **+0.08% Funding Arb over 24H** and **−0.12% over 7D**. Those illustrative numbers say that the latest day favored the displayed direction while the full week went the other way. Open the 7D charts to see when funding changed and whether both venues have enough history to compare. Then use the calculator to compare estimated fees and projected return at your intended holding period.

![Arbitrage calculator showing capital and holding period inputs and estimated net profit, APR, and ROI](images/profit-calculator.png)

**Net Profit** is the estimated dollar result for the selected hold. **Net ROI** expresses that result as a percentage of the entered capital. **Net APR** expresses it on an annualized basis. The breakdown shows the estimated trading fees and price basis separately.

## Spread Tracking

![BTC price comparison on Hyperliquid and RiseX in dark mode](images/spread-tracking-btc.png)

| Chart | What to look for |
| --- | --- |
| **Price Arbitrage** | Compare the two price lines. Look for when the price gap widened or narrowed across the two venues. |
| **Funding Arbitrage** | Compare quoted funding over time. Watch for brief spikes, sustained differences, and reversals. |
| **Funding Payment History** | Read net settled funding for the displayed direction, calculated as short-leg funding minus long-leg funding. Positive bars favor that direction; negative bars work against it. A bucket needs observations from both legs. |

Start with **24H** to inspect recent moves. Use **3D** or **7D** to see whether the pattern repeats. **Shared coverage** measures how much of the selected window has observations from both legs. A line can connect observations across a missing interval, so a continuous line alone does not mean coverage is complete. [Open Spread Tracking](https://yieldmaxi.xyz/tools/spread-tracking)

![Funding chart showing the two BTC venues' quoted rates over 24 hours](images/spread-tracking-funding.png)

*Funding Arbitrage shows when the venues quoted different rates during the day.*

![Green and red bars showing net settled funding for the BTC long and short legs](images/spread-tracking-payments.png)

*Funding Payment History shows net settled funding by time bucket. Green bars are positive, red bars are negative. Hover a bar to inspect both legs and the net value.*

## Delta Neutral

In **Delta Neutral**, choose **Same-Exchange** to see spot and perp legs on one venue, or **Cross-Exchange** to compare routes that use two venues. Pick a historical window and compare **Net APR**, **Coverage**, and liquidity across routes. Cross-exchange routes also require you to consider where collateral is held and how funds can move between the venues. [Open Delta Neutral](https://yieldmaxi.xyz/delta-neutral)

### Same-Exchange

![Same-Exchange Delta Neutral table with asset, coverage, and Net APR columns](images/delta-same.png)

| Column | What it tells you |
| --- | --- |
| **Rank** | The route's position under the current sort. The star adds it to your watchlist. |
| **Asset** | The underlying asset and the exchange used for both the spot and perpetual legs. |
| **Volume (Avg)** | Average spot trading volume for the selected window. |
| **Open Interest (Avg)** | Average open interest for the perpetual market over the selected window. |
| **Coverage** | The share of the selected period backed by observations. |
| **Funding** | Funding for the selected period, expressed as a percentage. Positive funding is paid by longs to shorts. |
| **APR** | Gross annualized funding yield before modeled trading costs. |
| **Fees** | Estimated trading fees for the spot and perpetual legs. |
| **Bid-Ask Spread** | Estimated execution cost from the bid–ask gaps in the two markets. |
| **Basis** | The price difference between the perpetual and spot legs. |
| **Net APR** | Estimated annualized return after costs and basis. The chart and calculator icons give more context for a route. |
| **Break-Even** | Estimated days of funding income required to recover modeled trading costs. |
| **Action** | Opens the exchange for the selected route. |

### Cross-Exchange

![Cross-Exchange Delta Neutral table with separate spot and perpetual venues](images/delta-cross.png)

| Column | What it tells you |
| --- | --- |
| **Rank** | The route's position under the current sort. |
| **Asset** | The underlying asset shared by the spot and perpetual legs. |
| **Spot Exchange** | The venue used to buy the asset. |
| **Spot Volume** | Trading volume on the spot leg. |
| **Perp Exchange** | The venue used to short the perpetual. |
| **Perp Open Interest** | Open interest on the perpetual leg. |
| **Coverage** | The share of the selected period with qualifying observations for this route. |
| **Funding 8H** | The perpetual leg's funding expressed on an eight-hour basis. The smaller **Σ** value estimates cumulative funding for the selected window. |
| **APR** | Gross annualized funding yield before modeled trading costs. |
| **Basis** | The price difference between the perpetual on one venue and spot on the other. |
| **Net APR** | Estimated annualized return after costs and basis. Select the value for APR history or use the calculator beside it. |
| **Fees** | Estimated round-trip fees for both legs. The smaller line shows the estimated spread cost separately. |
| **Break-Even** | Estimated days of funding income required to recover the modeled trading costs. |
| **Action** | Opens the spot and perpetual venues through separate links. |

## Stats

![Market Overview with a fear and greed gauge, historical readings, and a sentiment chart](images/market-overview.png)

**Market Sentiment** places the latest Fear & Greed score alongside recent readings and a historical chart. The scale runs from 0 (extreme fear) to 100 (extreme greed).

![Stablecoin Liquidity charts comparing market capitalization for USDT and USDC](images/stablecoin-liquidity.png)

**Stablecoin Liquidity** tracks stablecoin market capitalization and **USDT Dominance**, the share of the overall crypto market capitalization represented by USDT. Use longer windows to put a daily change in context. [Open Stats](https://yieldmaxi.xyz/market) · [USDT Dominance](https://www.tradingview.com/symbols/USDT.D/)

### Exchanges

![Exchanges table with volume, open interest, fees, and execution cost estimates at three order sizes](images/exchanges.png)

| Column | What it tells you |
| --- | --- |
| **#** | The venue's position in the current sort. |
| **Name** | The exchange being compared. |
| **Volume (24H)** | The USD value traded in tracked perpetual markets over the last 24 hours. Select a value to view its history when available. |
| **Open Interest** | The USD value of open perpetual positions. Select a value to view its history when available. |
| **Maker Fee** | The displayed rate for a fill that adds liquidity. Account terms may change the actual fee. |
| **Taker Fee** | The displayed rate for a fill that removes liquidity. |
| **Reference Contract** | The specific contract used to estimate execution costs for that venue. |
| **Execution Cost $10K** | Estimated one-way cost in USD for a $10,000 order on the reference contract. |
| **Execution Cost $100K** | Estimated one-way cost in USD for a $100,000 order on the same reference contract. |
| **Execution Cost $1M** | The same comparison at $1 million. An estimate may be unavailable where a fill at this size cannot be assessed. |

Check the estimate's tooltip for its cost basis. Compare the same reference contract and order size when you can.

## Lending

![Lending table comparing assets, chains, pool size, and base and reward APY](images/lending.png)

Compare pools for the same asset across chains. A pool can have a higher **Total APY** because of temporary incentives, so read **Base APY**, **Reward APY**, and **APY (30D avg)** together. [Open Lending](https://yieldmaxi.xyz/lending)

| Column | What it tells you |
| --- | --- |
| **Rank** | The pool's position under the current sort. The star adds it to your watchlist. |
| **Protocol** | The lending platform that hosts the pool. |
| **Chain** | The blockchain network on which the pool operates. |
| **Asset** | The token supplied to the pool. |
| **TVL** | Total USD value currently supplied to the pool. |
| **Total APY** | Current annual percentage yield from base interest and rewards, where applicable. |
| **Base APY** | The portion of yield from lending activity. |
| **Reward APY** | The portion of yield from incentives. |
| **APY (30D avg)** | Average reported APY over the past 30 days. |
| **Action** | Opens the pool or protocol page. |

## Vaults

![Vaults table showing deposit limits, lockup, APY, and notices](images/vaults.png)

Start with the deposit asset and lockup period, then use **Notice** to see how the vault works and what can affect the return. [Open Vaults](https://yieldmaxi.xyz/vaults)

| Column | What it tells you |
| --- | --- |
| **Rank** | The vault's position under the current sort. The star adds it to your watchlist. |
| **Protocol** | The venue or protocol operating the vault. |
| **Asset** | The token accepted for deposits. |
| **TVL** | Total USD value held in the vault. |
| **Minimum Deposit** | The smallest deposit the vault accepts. |
| **Maximum Capacity** | The published deposit cap or vault capacity, depending on how the product reports its limit. |
| **Lockup Period** | The reported lockup or cooldown that applies before a withdrawal completes. |
| **APY** | Current reported annual percentage yield. An interactive value opens APY history where available. |
| **Notice** | Opens details about the strategy, restrictions, and risks. |
| **Action** | Opens the provider's vault page. |

## Airdrop

![Airdrop program cards with potential ratings, earning instructions, and timelines](images/airdrop.png)

Use the category filters to switch between perpetual DEX, HIP3 Ecosystem, and DeFi Protocol programs. Each card summarizes what participants can do and the details available for the program. [Open Airdrop](https://yieldmaxi.xyz/airdrop)

| Card field | What it tells you |
| --- | --- |
| **Program name and description** | Identifies the project and summarizes the campaign. |
| **Estimated potential** | A qualitative High, Medium, or Low rating. Actual eligibility, allocation, and payouts depend on program terms. |
| **How to Earn** | The activity the program lists for participation, such as trading or providing liquidity. |
| **Timeline** | The published timing when available. |
| **Funding and backers** | Disclosed fundraising and investor information for the project. |
| **Join & Farm Points** | Opens the project or program page so you can check its current terms. |

## Chart

![Chart workspace displaying multiple symbol panels in one layout](images/chart.png)

Use the separate **Chart** workspace for side-by-side price analysis. Choose a **Layout** for the number of panels you want and set the **Symbol** in each panel. Your layout is saved in the current browser. For funding quotes or settlement history for a specific pair, use Spread Tracking. [Open Chart](https://yieldmaxi.xyz/tools/chart)

## Read the numbers

| Metric | How to interpret it |
| --- | --- |
| **Net APR** | An annualized estimate based on the displayed window and cost assumptions. A short window can make a brief funding spike look large when annualized. |
| **Coverage** | How much of the selected period has observations. Compare results over the same window and check both legs when coverage is uneven. |
| **Volume and Open Interest** | Volume measures recent trading activity. Open interest measures positions still open. Both help assess market activity, while execution also depends on the size of your order and available liquidity. |
| **Break-even** | The estimated time for funding income to offset the displayed trading costs under the current assumptions. A move in funding or execution prices can change it. |
| **A dash or blank value** | The figure is unavailable for that pair and period. A displayed `0` means the available data supports a zero value at the shown precision. |

For background on perpetuals, funding, and other yield strategies, see the product guide. [Yield Maxi product guide](https://yieldmaxi.xyz/guide)
