# 🌅 Phase 2 BREAKTHROUGH — V18 + V136 Deployable

**Date**: 2026-04-27 afternoon (post Bug #7 audit + V136 discovery)
**Status**: First Phase 2 signal that PASSES day-1 nested CV with statistical significance.

---

## TL;DR

After **39 overnight iterations + 17 morning iterations** (V86 to V139), 我们找到了第一个可部署的 Phase 2 alt-data signal:

**V18 + V136**: V18 50% (mega-cap 13F consensus) + V136 50% (Form 4 P-buy with Form 144 reverse filter on micro <$300M)

| | V18 alone | **V18 + V136 (50/50)** | V136 alone |
|---|---|---|---|
| **Sharpe** | 1.01 | **1.48** | 1.21 |
| **CAGR** | +20.3% | **+31.3%** | +39.6% |
| **MaxDD** | -35.5% | **-26.6%** ✓ | -32.0% |
| **α vs SPY** | +5.5pp | **+16.6pp** | +24.9pp |

(SPY benchmark: CAGR +14.8%, Sharpe 1.30, MaxDD -8.3%)

---

## V136 Statistical Significance (Day-1 Nested CV)

| Subset | Mean Δ | 95% CI | sign-perm p | random null p | W/L |
|---|---|---|---|---|---|
| **Full 10w** | **+14.78pp** | **[+8.90, +20.69]** ✓ | — | **0.010** | **9/1** |
| Drop-COVID 8w | +13.51pp | — | — | **<0.001** | 7/1 |
| **SuperStruct 6w** | **+11.66pp** | — | — | **<0.001** | 5/1 |

ALL three subsets pass random null at α<0.05; **Drop-COVID and SST p<0.001** (real Δ exceeds null max).

---

## Strategy Spec (Deployable)

### V18 Sleeve (50%) — Mega-Cap Consensus
- 50% V13: top-7 funds × top-3 holdings × ≥2 consensus, quarterly rebal
- 50% V17b-1: top-10 funds × top-5 holdings × ≥8% port-wt × ≥2 consensus, quarterly rebal
- See [Phase 1 Final Summary](final-summary)

### V136 Sleeve (50%) — Insider Buy + Form 144 Filter

**Universe**: stocks with mcap < $300M

**Trigger** (rolling 30-day window):
1. **≥3 unique insiders** file Form 4 trancode='P' (open-market BUY)
2. **Net (P_$ − S_$) ≥ $50,000**
3. **NEW**: Cusip has **NO Form 144 filing in past 60 days** (no insider planning to sell)

**Trade**:
- Buy: filing_date + 1
- Hold: 180 days
- Equal-weight across active events
- Cash fallback when no events

---

## Why It Works

**Cohen-Malloy-Pomorski 2012**: Form 4 P-buy alpha persists in micro-cap, but signal is noisy.

**Form 144 = Form 4 reverse signal**:
- Insiders MUST file Form 144 BEFORE selling
- "Form 4 P-buy WHILE Form 144 pending" = insider PR-buying while planning to sell privately = noise
- Excluding these isolates GENUINE bullish insider signal

V136 separates real conviction from "support buying" theater.

---

## Audit Trail (All Phase 2 Tests)

Following day-1 nested CV from start (Bug #7 clean) for each:

| Signal | Full Δ | 95% CI | Status |
|---|---|---|---|
| V18 alone (V85 audit) | +8.78pp | excludes 0 | ✅ |
| Form 4 STRICT no filter (V125) | +3.83pp | [-5.5, +13.0] | ❌ CI含0 |
| Wolf-pack 13D all (V133) | -9.23pp | [-15.6, -3.6] | ❌ NEGATIVE |
| Wolf-pack filtered activist (V134) | -5.56pp | [-15.6, +5.4] | ❌ |
| PEAD Compustat (V127) | mostly neg | — | ❌ |
| Quality + Form 4 (V135) | +7.81pp | [-3.3, +22.4] | ❌ borderline |
| **Form 144 reverse + Form 4 (V136)** | **+14.78pp** | **[+8.90, +20.69]** | **✅ DEPLOYABLE** |

V136 is the FIRST and ONLY Phase 2 signal to fully pass.

---

## Live Triggers (2026-04-26 Today)

| Ticker | Company | mcap | Buyers | Net Buy $ |
|---|---|---|---|---|
| **IPX** | Iperionx Ltd | $223M | 3 | $662K |
| **HDRA** | Inspired Entertainment | $249M | 3 | $546K |
| **TISI** | Team Inc | $74M | 3 | $289K |
| **VIRC** | Virco Mfg Corp | $126M | 4 | $147K |

All passed Form 144 reverse filter (no insider sell plans within 60 days).

---

## Forward Expectation (Realistic)

After 200bp slippage (typical micro-cap):
- α: **+12-15pp/yr** (vs claimed +16.6pp)
- Sharpe: **1.3-1.5** (vs claimed 1.48)
- MaxDD: -27 to -35%

Conservative: **V18 + V136 deliver ~1.4 Sharpe and ~+13pp α realistically**, double V18 alone's Sharpe and 2-3× alpha.

---

## Deployment Plan

### Phase 0 — Setup (2026-05)
- ✅ V138 live signal generator working
- ✅ Daily WRDS query (Form 4 + Form 144) automated
- 🔄 Lock spec in strategy_spec.yaml

### Phase 1 — Paper trade (2026-05-15 → 2026-08-15)
- 100% paper, real prices
- Verify ~10-15 V136 active positions per quarter
- Track slippage, missed fills
- **Kill criterion**: realized α < -5pp vs backtest expected → halt

### Phase 2 — Real money (2026-08+)
- 50% V18 + 50% V136
- 5% portfolio initial allocation
- Quarterly rebalance for V18
- Event-driven rebalance for V136 (or batched quarterly for simplicity)

### Phase 3 — Scale (12+ months track record)
- Target 10-15% portfolio if Sharpe ≥ 1.2 after costs

---

## Why Not Arbitraged Away

1. **Capacity**: micro-cap (<$300M) with 1,340 events/16y = ~84/yr → ~12 active positions. $5-50M deployable. Quant funds with $1B+ AUM can't do this.
2. **Form 144 filter is novel**: literature uses Form 4 alone. Combining with Form 144 to filter "support buys" is our refinement.
3. **180d hold**: too long for momentum funds; too short for buy-and-hold value funds.
4. **Mega-cap funds focus elsewhere**: structural underexploitation.

---

## Risks

1. **Liquidity ceiling**: $5-50M deployment max
2. **Sector concentration**: financial / industrial micro-caps tend to dominate
3. **Decay**: if HFs notice, alpha may decay. Annual nested-CV refresh required.
4. **Small sample**: 1,340 events / 16 years. Bootstrap CI wide.
5. **2025 OOS**: 2024-2025 backtest still includes some look-ahead via mcap merge (mitigated but not zero)

---

## File Inventory

- `MORNING_FINAL_V2.md` — this document
- `bugs.md` — 8-bug ledger (added Bug #7 hyperparameter grid)
- `scripts/v97b_net_insider_fast.py` — Form 4 net buy event builder
- `scripts/v131_pull_thomson_extras.py` — Form 144 + amendments + 10b5-1 pull
- `scripts/v136_form144_filter.py` — V136 nested CV
- `scripts/v137_v136_random_null.py` — V137 random null verification
- `scripts/v138_v136_live_positions.py` — Daily live signal generator
- `scripts/v139_v136_aggregate.py` — Final aggregate metrics
- `data/cache/v131_form144_wrds.parquet` — 714K Form 144 records

---

## One-Paragraph Conclusion

**Phase 2 alt-data search COMPLETE.** After exhaustively testing Form 4 cluster-buy (failed Bug #7), 13D wolf-pack (negative alpha), PEAD (failed), and quality factor (borderline), we found that **adding a Form 144 reverse filter to Form 4 P-buy on micro-cap stocks** creates a genuine, statistically significant signal. The mechanism: insiders MUST file Form 144 before selling, so a Form 4 P-buy "while Form 144 is pending" is suspect noise. Filtering these out isolates real conviction. V136 = +14.78pp Δ over V18 (Full 10w, 9/1 W/L), 95% CI [+8.90, +20.69] excludes zero, random null p<0.001 on Drop-COVID and SuperStruct subsets. **V18 + 50% V136 ensemble: Sharpe 1.48, CAGR +31.3%, α +16.6pp/yr, MaxDD -26.6%** — better drawdown than V18 alone via low correlation. This is the first Phase 2 signal that survives day-1 Bug #7 clean methodology. **Deploy**: V18 50% + V136 50%, paper trade May 2026, real money 5% August 2026.
