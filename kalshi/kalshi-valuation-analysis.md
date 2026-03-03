---
layout: default
title: Kalshi Valuation Analysis
---

# Kalshi Valuation Analysis

*Last updated: February 22, 2026*

## Executive Summary

Kalshi is the first CFTC-regulated prediction market exchange in the US. At an $11B valuation (Series E, Dec 2025), the market is pricing in continued hyper-growth. This analysis uses **actual trading volume data** from our database (which captures ~46% of Kalshi's total volume) calibrated against Kalshi's **reported 2025 revenue of $263.5M** to project 2026 revenue and earnings.

**Key findings:**
- 2026 projected revenue: **$1.3B - $1.9B** (depending on growth assumptions)
- At mid-range costs ($180M/yr), all scenarios are profitable
- At $11B valuation: **5.7-8.4x forward revenue**, **6.3-9.8x forward earnings**
- Biggest risk: Robinhood building its own exchange (currently drives >50% of Kalshi's volume)

---

## 1. Company Overview

| Metric | Value |
|--------|-------|
| **Valuation** | $11B (Series E, Dec 2025) |
| **Total funding** | ~$1.7B |
| **2025 fee revenue** | $263.5M |
| **2025 trading volume** | $22.88B |
| **YoY revenue growth** | ~900% (2024 to 2025) |
| **Sports % of revenue** | 89% |
| **Regulatory status** | CFTC Designated Contract Market (DCM) |
| **Profitable?** | No (as of Nov 2025) |

### Funding History

| Round | Date | Amount | Valuation | Lead |
|-------|------|--------|-----------|------|
| Series D | Oct 2025 | $300M | $5B | a16z, Sequoia |
| Series E | Dec 2025 | $1B | $11B | Paradigm |

Valuation more than doubled in under two months.

---

## 2. Fee Structure

Kalshi earns fees on matched trades without taking positions against users.

| Fee Type | Formula | Description |
|----------|---------|-------------|
| **Taker** | `0.07 x contracts x P x (1-P)` | Standard fill fee |
| **Maker** | `0.0175 x contracts x P x (1-P)` | 75% discount for liquidity providers |
| **Settlement** | $0 | No fee on resolution payouts |

### Fee at Different Price Points

| Contract Price | Taker Fee | Maker Fee |
|:--------------:|:---------:|:---------:|
| $0.50 | 1.75% | 0.44% |
| $0.90 | 0.63% | 0.16% |
| $0.95 | 0.33% | 0.08% |
| $0.99 | 0.07% | 0.02% |

The average effective fee rate across all 2025 trades was approximately **1.15%** of volume ($263.5M / $22.88B).

---

## 3. Volume Growth (Actual Data)

Our database tracks settled markets on Kalshi, capturing approximately **46% of total platform volume**. Below is the actual monthly data:

| Month | Markets Settled | Contracts (our DB) | Est. Revenue | MoM Growth |
|:-----:|:---------:|:-----------:|:----------:|:----------:|
| Jan 2025 | 13,391 | 1.49B | $16.5M | -- |
| Feb 2025 | 13,382 | 289M | $3.2M | -81%\* |
| Mar 2025 | 15,381 | 471M | $5.2M | +63% |
| Apr 2025 | 16,531 | 388M | $4.3M | -18% |
| May 2025 | 19,038 | 665M | $7.3M | +71% |
| Jun 2025 | 17,120 | 845M | $9.3M | +27% |
| Jul 2025 | 19,137 | 729M | $8.1M | -14% |
| Aug 2025 | 21,236 | 721M | $8.0M | -1% |
| **Sep 2025** | **25,794** | **2.78B** | **$30.8M** | **+286%** |
| Oct 2025 | 42,706 | 4.05B | $44.7M | +45% |
| Nov 2025 | 73,210 | 5.46B | $60.3M | +35% |
| Dec 2025 | 77,614 | 5.96B | $65.8M | +9% |
| **Jan 2026** | **120,927** | **8.21B** | **$90.7M** | **+38%** |
| Feb 2026 | 356,906 | ~8.04B\*\* | ~$88.8M | -2% |

\* *Jan 2025 inflated by 2024 election market settlements. Feb 2025 is the true pre-sports baseline.*
\*\* *Feb 2026 extrapolated from 22 days of data to full 28-day month.*

### Revenue Calibration

Revenue estimates are calibrated against Kalshi's reported 2025 figures:

```
Reported 2025 revenue:     $263.5M
Our DB 2025 contracts:     23.85B
Implied fee/contract:      $0.01105 (per our-DB contract)
DB coverage factor:        ~46% of total Kalshi volume
```

This rate is applied consistently to project future revenue from observed contract volumes.

### Key Growth Inflections

1. **Sep 2025 (+286%)**: Kalshi launched sports event contracts on Jan 24, 2025, but volume exploded with the NFL season in September
2. **Oct-Dec 2025 (+9% to +45%)**: Sustained sports-season growth
3. **Jan 2026 (+38%)**: Super Bowl anticipation + continued platform adoption
4. **Feb 2026 Super Bowl**: Single-day volume of $871M on Super Bowl Sunday

### YoY Growth

| Metric | Feb 2025 | Feb 2026 | YoY |
|--------|:--------:|:--------:|:---:|
| Markets settled | 13,382 | 356,906 | **27x** |
| Contracts | 289M | ~8.0B | **28x** |

---

## 4. Growth Trend Model

### Methodology

Rather than using arbitrary growth assumptions, we derive projections from **actual observed patterns**:

1. **Fee rate**: Calibrated from 2025 reported revenue ($263.5M / 23.85B our-DB contracts = $0.01105/contract)
2. **Baseline**: Average of Jan-Feb 2026 actual volume (~8.1B contracts/month in our DB)
3. **Growth rates**: Three scenarios based on observed post-sports-launch MoM growth:
   - **Conservative**: 5% MoM compound growth
   - **Base**: 10% MoM compound growth
   - **Bull**: 12% MoM (half of the 24% geometric mean observed Oct 2025 - Feb 2026)
4. **Seasonality**: Derived from observed volume patterns — sports (89% of revenue) drives strong Sep-Feb peaks and Mar-Aug troughs

### Seasonal Multipliers

Based on observed contract volume patterns since Kalshi's sports launch:

| Period | Months | Multiplier | Driver |
|--------|--------|:----------:|--------|
| Super Bowl peak | Jan-Feb | 1.4x | NFL playoffs, Super Bowl |
| Off-season trough | Mar-Aug | 0.65-0.73x | No NFL; MLB, NBA lower volume |
| NFL return | Sep-Oct | 1.1-1.3x | NFL regular season begins |
| NFL peak | Nov-Dec | 1.3x | NFL + holiday engagement |

*Sports = 89% of Kalshi revenue. Non-sports (11%) is treated as flat year-round.*

---

## 5. 2026 Revenue Projections

### Conservative Scenario (5% MoM Growth)

| Month | Est. Revenue | Cumulative | Note |
|:-----:|:-----------:|:----------:|:----:|
| Jan | $90.7M | $90.7M | Actual |
| Feb | $88.8M | $179.5M | Actual |
| Mar | $68.3M | $247.8M | Projected |
| Apr | $65.2M | $313.1M | Projected |
| May | $68.5M | $381.6M | Projected |
| Jun | $79.1M | $460.7M | Projected |
| Jul | $75.5M | $536.2M | Projected |
| Aug | $87.2M | $623.4M | Projected |
| Sep | $141.5M | $764.9M | Projected |
| Oct | $166.1M | $931.0M | Projected |
| Nov | $183.6M | $1,114.6M | Projected |
| Dec | $192.8M | $1,307.4M | Projected |
| **Full Year** | **$1,307M** | | |

### Base Scenario (10% MoM Growth)

| Month | Est. Revenue | Cumulative | Note |
|:-----:|:-----------:|:----------:|:----:|
| Jan | $90.7M | $90.7M | Actual |
| Feb | $88.8M | $179.5M | Actual |
| Mar | $71.6M | $251.1M | Projected |
| Apr | $71.6M | $322.7M | Projected |
| May | $78.8M | $401.4M | Projected |
| Jun | $95.3M | $496.7M | Projected |
| Jul | $95.3M | $592.0M | Projected |
| Aug | $115.3M | $707.3M | Projected |
| Sep | $196.0M | $903.3M | Projected |
| Oct | $241.0M | $1,144.3M | Projected |
| Nov | $279.0M | $1,423.4M | Projected |
| Dec | $306.9M | $1,730.3M | Projected |
| **Full Year** | **$1,730M** | | |

### Bull Scenario (12% MoM Growth)

| Month | Est. Revenue | Cumulative | Note |
|:-----:|:-----------:|:----------:|:----:|
| Jan | $90.7M | $90.7M | Actual |
| Feb | $88.8M | $179.5M | Actual |
| Mar | $72.8M | $252.3M | Projected |
| Apr | $74.0M | $326.2M | Projected |
| May | $82.7M | $408.9M | Projected |
| Jun | $101.7M | $510.6M | Projected |
| Jul | $103.4M | $614.0M | Projected |
| Aug | $127.1M | $741.2M | Projected |
| Sep | $219.7M | $960.9M | Projected |
| Oct | $274.5M | $1,235.4M | Projected |
| Nov | $323.1M | $1,558.5M | Projected |
| Dec | $361.3M | $1,919.7M | Projected |
| **Full Year** | **$1,920M** | | |

### Revenue Summary

| Scenario | 2025 Actual | 2026 Projected | YoY Growth |
|----------|:-----------:|:--------------:|:----------:|
| Conservative (5% MoM) | $263.5M | **$1,307M** | +396% |
| Base (10% MoM) | $263.5M | **$1,730M** | +557% |
| Bull (12% MoM) | $263.5M | **$1,920M** | +629% |

---

## 6. Earnings Projections

### Cost Assumptions

Kalshi's cost structure is not publicly disclosed. We model three scenarios:

| Scenario | Annual Cost | Monthly | Includes |
|----------|:----------:|:-------:|----------|
| **Low** | $120M | $10M | Lean team (~100), minimal legal spend |
| **Mid** | $180M | $15M | ~200 employees, moderate legal |
| **High** | $250M | $21M | Full team (~250), heavy legal (19 lawsuits) |

### Earnings Matrix (Revenue - Costs)

| | Low Costs ($120M) | Mid Costs ($180M) | High Costs ($250M) |
|---|--:|--:|--:|
| **Conservative ($1,307M rev)** | $1,187M (91%) | $1,127M (86%) | $1,057M (81%) |
| **Base ($1,730M rev)** | $1,610M (93%) | $1,550M (90%) | $1,480M (86%) |
| **Bull ($1,920M rev)** | $1,800M (94%) | $1,740M (91%) | $1,670M (87%) |

*Percentages show net margin.*

All scenarios are **highly profitable** due to exchange economics — near-zero marginal cost per incremental trade.

---

## 7. Valuation Multiples

### At Current $11B Valuation

| Metric | Conservative | Base | Bull |
|--------|:-----------:|:----:|:----:|
| **EV / 2026 Revenue** | 8.4x | 6.4x | 5.7x |
| **EV / 2026 Earnings** (mid cost) | 9.8x | 7.1x | 6.3x |
| **EV / 2025 Revenue** | 41.7x | 41.7x | 41.7x |

### Comparable Companies

| Company | EV | Revenue | EV/Rev | EV/EBITDA | Growth |
|---------|:--:|:------:|:------:|:---------:|:------:|
| **Flutter (FanDuel)** | ~$69B | ~$14.5B | 4.8x | ~12x | ~15% |
| **DraftKings** | ~$31B | ~$5.5B | 5.6x | ~18x | ~20% |
| **Kalshi** | $11B | $1.3-1.9B\* | 5.7-8.4x | 6-10x | ~400-600% |
| **Polymarket** | ~$11.6B | N/A | N/A | N/A | N/A |

\* *2026 projected*

### Interpretation

- On **trailing 2025 revenue** ($263.5M), Kalshi trades at **42x** — an extreme premium
- On **forward 2026 revenue**, the multiple compresses to **5.7-8.4x** — comparable to DraftKings
- The premium over Flutter/DraftKings is justified by:
  - 400-600% YoY growth vs 15-20%
  - Exchange model (near-zero marginal costs) vs sportsbook model (risk-bearing)
  - Regulatory moat as only CFTC-licensed prediction market with sports
  - Optionality in non-sports categories (politics, crypto, weather)

---

## 8. Key Risks

### 1. Robinhood Disintermediation (Critical)

Robinhood currently drives **>50% of Kalshi's volume** through a distribution partnership. In January 2026, Robinhood acquired MIAXdx, a CFTC-licensed exchange, via joint venture with Susquehanna. They are likely building their own prediction market platform. If Robinhood migrates volume in-house, Kalshi could lose half its revenue overnight.

**Impact**: Revenue could fall 40-60% in a worst case.

### 2. State Regulatory Battles (High)

Kalshi faces **19 active federal lawsuits** from state gaming commissions and tribal authorities:
- Massachusetts: Preliminary injunction — sports contracts blocked without state license
- Nevada, Connecticut, Tennessee: Various ongoing challenges
- Consumer class action filed Nov 2025

**Impact**: Could be blocked from key states, fragmenting the market.

### 3. Sports Concentration (Medium)

89% of revenue comes from sports markets. A single category driving ~90% of revenue creates:
- **Seasonal risk**: Mar-Aug revenue drops ~50% vs peak months
- **Regulatory risk**: Sports betting faces stricter state scrutiny than other categories
- **Competitive risk**: Sports betting is a crowded market (Flutter, DraftKings, BetMGM)

### 4. Fee Compression (Medium)

As competition increases (Robinhood, Polymarket, Gemini Titan), Kalshi may face pressure to lower fees. Current average ~1.15% effective rate could compress toward 0.5-0.8%.

### 5. Not Yet Profitable (Low-Medium)

Despite $263.5M in 2025 revenue, Kalshi was not profitable as of November 2025. Legal costs from 19 lawsuits + rapid hiring + infrastructure spending likely consume significant capital.

---

## 9. Scenario Analysis

### Bear Case: Robinhood Exits + Regulatory Headwinds

- Lose 50% of volume when Robinhood launches own exchange (H2 2026)
- Fee compression from competition (1.15% → 0.8%)
- Blocked in 3-5 key states
- **2026 revenue: ~$500-700M**, still profitable but growth narrative breaks
- **Implied EV/Rev: 16-22x** — too expensive, valuation compresses

### Base Case: Continued Growth with Some Headwinds

- Robinhood transition is gradual (lose 20-30% over 12 months)
- Kalshi expands non-sports categories to reduce concentration
- Regulatory outcomes mixed (win some, lose some)
- **2026 revenue: $1.3-1.7B**, strong profitability
- **Implied EV/Rev: 6-8x** — fairly priced for growth

### Bull Case: Network Effects Dominate

- Kalshi retains volume despite Robinhood due to brand + liquidity moat
- Sports betting regulation clarified in Kalshi's favor
- Non-sports categories (crypto, politics, weather) grow to 20%+ of revenue
- **2026 revenue: $1.7-2.0B**, margins above 85%
- **Implied EV/Rev: 5.5-6.5x** — cheap for a high-growth exchange

---

## Methodology Notes

### Data Sources
- **Trading volume**: Our `kalshi_history.db` database of settled markets (captures ~46% of total Kalshi volume)
- **Revenue calibration**: Kalshi's reported 2025 fee revenue of $263.5M (public filings)
- **Fee structure**: Kalshi help center documentation
- **Funding/valuation**: Public announcements (TechCrunch, BusinessWire)
- **Comparable data**: Public company filings (Flutter, DraftKings)

### Revenue Estimation Method
1. Sum contract volume from our database by month
2. Compute implied fee per contract: `$263.5M / 23.85B contracts = $0.01105`
3. Apply this rate to monthly volumes for revenue estimates
4. For projections: compound growth from Jan-Feb 2026 baseline, adjusted for sports seasonality
5. Seasonal multipliers derived from observed Sep 2025 - Feb 2026 volume patterns

### Limitations
- Our DB captures ~46% of volume — coverage ratio may shift over time
- Fee-per-contract assumes constant mix of taker/maker and price distribution
- Seasonal patterns based on only one sports cycle (Sep 2025 - Feb 2026)
- Projections assume no structural breaks (Robinhood exit, fee changes, regulation)
- Cost estimates are speculative — Kalshi does not disclose expenses

---

*Analysis by Kalshi Bond Grinding Bot research team. Data as of February 22, 2026.*
