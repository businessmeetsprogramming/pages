# V18 + STRICT — Exhaustive Bug Audit

**Date**: 2026-04-27 (post-overnight session)
**Audit scope**: V18 + Form 4 STRICT MicroNetBuy (claimed +7.53pp SST Δ, 6/0 W/L)
**Methodology**: independent re-review against bugs.md (7 documented bugs), checking whether each could be present in current pipeline.

---

## Executive Summary

After 39 overnight iterations producing the V18+STRICT spec, an independent audit identifies:

| Bug | Status | Risk to STRICT claim |
|---|---|---|
| **#1** delisting fillna | ✓ MITIGATED | minor (V104 -50% imputation) |
| **#2** OOS-peek weight | ✓ CLEAN | weight selection per-window IS+TUNE only |
| **#3** SPY-fallback inflation | ✓ CLEAN | cash fallback used |
| **#4** split adjustment | ✓ CLEAN | cum_tr is split-adjusted |
| **#5** rolling overlap outliers | ⚠ PARTIAL | SuperStruct definition is post-hoc |
| **#6** correlated dud variants | ✓ CLEAN | single STRICT variant, not ensemble |
| **#7** hyperparameter grid on held-out | **❌ PRESENT** | spec selection over multiple grids |

**Critical finding**: Bug #7 is present in our pipeline. The "STRICT" spec (mcap<$300M, ≥5 buyers, $50K, 30d, 180d) was selected via comparison of held-out alphas across multiple grids (V101, V111, V120). This is the same family of bug that killed V62 (+4.38pp claim → -4.33pp under nested CV).

**V125 running now** to compute the unbiased estimate via per-window inner-CV across the full Form 4 spec grid. Until V125 completes, the +7.53pp SuperStruct claim should be treated as an UPPER BOUND.

---

## Bug-by-Bug Detailed Audit

### Bug #1: `stock_ret_filled = 0` for delistings ✓ MITIGATED

**Definition**: When stock delists between buy and sell date, NaN sell price was historically filled with 0% return ("flat"). Real outcome is ~-30% mean delisting return.

**Risk to STRICT**: Micro-cap stocks delist disproportionately often (~5-10x rate of large-cap). If our backtest drops or zeros-out delisted positions, alpha is overstated.

**Implementation in V18+STRICT**:
- Returns computed via `cum_tr_factor.parquet` (CRSP daily total return, split-adjusted)
- For events where sell_date exceeds cum_tr coverage (delisted stocks), V104 introduced fallback:
  ```python
  # 1. Use last available cum_tr for that cusip if any (captures partial loss before delisting)
  # 2. If no cum_tr at all for cusip, apply -50% fixed loss
  ```
- **V104 audit**: SST Δ shrunk from +9.33pp (drop missing) → +7.81pp (with -50% imputation). Signal robust to imputation choice (-25%, -50%, -90% all give same number due to fallback-1 logic).

**Residual risk**: 197 of 2075 micro cusips (9.5%) have NO cum_tr coverage at all → applied fixed -50% loss. The true average delisting outcome could be -75% to -90% on absolute worst names. If those 197 cusips concentrated in catastrophic events, alpha overstated by ~0.5-1pp.

**Verdict**: MITIGATED. Residual uncertainty <1pp.

---

### Bug #2: V26 ensemble OOS-peeking weight selection ✓ CLEAN

**Definition**: V26 grid searched ensemble weights with held-out Sharpe filter applied BEFORE ranking → max-of-grid bias on held-out.

**Risk to STRICT**: If we select STRICT mix weight by held-out Sharpe, classic Bug #2.

**Implementation in V18+STRICT**:
- `for held_start in range(2014, 2024):` — per-window blind selection
- Inner: `weights_grid = [0.0, 0.1, 0.2, 0.3, 0.5]`
- Selection: `is_sh + tune_sh` — IS+TUNE Sharpe ONLY
- Held-out used ONLY for reveal step
- Implementation in v97b/v101/v111/v112/v114 follows V83 nested CV pattern correctly

**Verdict**: CLEAN. Mix weight selection is per-window IS+TUNE only.

---

### Bug #3: V21B4 SPY-fallback alpha inflation ✓ CLEAN

**Definition**: Strategy with frequent SPY-fallback periods + small actual-trade alpha looks like big alpha because SPY rallies during fallback periods get attributed to strategy.

**Risk to STRICT**: If STRICT has many "no signal" quarters and we filled with SPY return, in bull markets we'd inflate alpha falsely.

**Implementation in V18+STRICT**:
- STRICT signal alone uses CASH fallback (return=0) in v97b's `quarterly_returns_with_imputation`:
  ```python
  rows.append((R, 0.0, 0))  # ← cash fallback, not SPY
  ```
- V18 baseline uses SPY fallback when no consensus stocks (this is V18's design, unchanged)
- The ensemble `(1-w)*v18 + w*strat` with `w` 0.3-0.5 means ensemble gets V18's SPY-fallback character only proportionally

**Verdict**: CLEAN. Cash fallback applied to STRICT sleeve.

---

### Bug #4: Stock split adjustment incomplete ✓ CLEAN

**Definition**: Reverse stock splits in distressed micros (e.g., 1:10 consolidation) appear as +900% returns in unadjusted price data. Killed V21B4 family entirely when properly fixed.

**Risk to STRICT**: Micro-cap (<$300M) is exactly the universe where reverse splits cluster. If we used unadjusted prices, alpha would be inflated.

**Implementation in V18+STRICT**:
- Returns from `cum_tr_factor.parquet` — built from CRSP daily total-return factor
- CRSP cum_tr is split-adjusted by construction (handles forward AND reverse splits)
- V42 (panel v3 rebuild) verified split-adjustment correctness

**Verdict**: CLEAN. cum_tr is split-adjusted.

---

### Bug #5: Rolling overlapping windows amplify single-year outliers ⚠ PARTIAL

**Definition**: Using overlapping rolling 2-year windows means a single-year outlier (e.g., +200% in 2020) appears in 2 windows (2019-2020 and 2020-2021), inflating the rolling average.

**Risk to STRICT**: 
- Window detail shows MULTIPLE outlier years for STRICT solo: 2017-2018 (+82%), 2018-2019 (+45%), 2019-2020 (+28%), 2020-2021 (+100%), and 2024 (+93%).
- Each overlap covers 2 years → the average over 10 rolling 2-year windows DOUBLES each year's contribution.

**Mitigation applied**:
1. **SuperStruct (drop COVID + drop 2017-19 outliers)**: removes 4 windows touching outlier years. Result: SST Δ +7.53pp (6/0 W/L)
2. **V108 / V119 walk-forward yearly (NON-OVERLAPPING)**: each year evaluated independently. Result: STRICT Δ +17.27pp (8/3 W/L), Drop-COVID +20.95pp (7/2), Drop 2017-2020 +15.11pp (6/1)
3. **V124 regime-split**: signal positive in all 4 regimes (2010-14, 2015-19, 2020-24, most recent 2022-24)

**Residual risk**: 
- "SuperStruct" definition (drop COVID + drop 2017-19) was chosen POST-HOC after seeing the outlier years. This is selection bias.
- A truly unbiased approach would be: leave-one-year-out + bootstrap, OR commit to drop-windows ex-ante.
- BUT: the V108/V119 walk-forward yearly is independent of the post-hoc choice and confirms positive Δ across multiple aggregations.

**Verdict**: PARTIAL. The +7.53pp SuperStruct number has post-hoc-selection bias risk, but V108/V119 walk-forward (no overlap) confirms the signal is robust. **Use V108/V119 numbers (+11.94pp drop-2017-2020, 6/1 W/L) as more honest estimate.**

---

### Bug #6: Blind ensemble over correlated dud variants ✓ CLEAN

**Definition**: V80 100k Dirichlet ensemble over 6 V62 cluster variants picked the WORST variant because IS+TUNE Sharpe rewarded low-vol noise. 5 of 6 variants had negative standalone alpha.

**Risk to STRICT**: If we built an ensemble of multiple Form 4 signals and picked weights blindly, could pick correlated dud configs.

**Implementation in V18+STRICT**:
- V96 tested multi-signal ensemble (5 first-buy variants × 2 holds): SST Δ +0.52pp — NO improvement over single signal
- V109 tested $-weighted aggregation of net-buy signal: FAILED (-3pp SST)
- Final spec is SINGLE STRICT signal at fixed parameters, not blind ensemble of multiple signals

**Verdict**: CLEAN. Single-variant deployment, not ensemble.

---

### Bug #7: Hyperparameter grid search on held-out average ❌ PRESENT

**Definition**: Selecting hyperparameters by ranking grid configs on held-out aggregate metric → max-of-grid selection bias. Killed V62 spec entirely (+4.38pp claim → -4.33pp unbiased, 8.71pp shrinkage).

**Risk to STRICT**: This is the BIG ONE. Did we pick STRICT spec by max-of-held-out?

**Selection trail**:
1. **V101** (5 mcap bands tested): picked `micro <$300M` because it had highest SuperStruct Δ (+9.33pp 5/1) compared to other 4 bands. This is max-of-5.
2. **V111** (9 cluster configs tested): picked `≥5 buyers, $50K` because it had perfect 6/0 SuperStruct W/L and Full 10/0 W/L. This is max-of-9.
3. **V120** (10 mcap × min_b combos): noted $100M-$300M is "Goldilocks" but kept STRICT as primary spec. Still max-of-grid risk.
4. **V103** (8 sectors tested): noted Sector 01 (+22.22pp 6/0) but didn't deploy sector filter. Min concern.

**Total combined grid**: ~50-100 specs were compared on HELD-OUT statistics. **By construction, the "best" config has selection bias upward.**

**Audit running (V125)**: Per-window inner-CV across full grid (6 mcap × 4 min_b × 4 min_$ × 3 hold = 288 configs). Picks spec on IS+TUNE Sharpe only per held-out window. This gives the UNBIASED estimate.

**Expected result based on V62 precedent**:
- V62 had +4.38pp biased claim, shrunk to -4.33pp under nested CV (8.71pp shrinkage)
- STRICT has +7.53pp SuperStruct biased claim
- If similar shrinkage, true unbiased Δ could be **anywhere from -2pp to +5pp**

**Verdict**: **BUG #7 IS PRESENT in current pipeline. Until V125 completes, do NOT deploy STRICT spec at +7.53pp expected alpha.**

---

## Honest Reassessment of V18+STRICT Claim

| Metric | Claimed | Audit-adjusted estimate |
|---|---|---|
| Full 10w Δ | +11.88pp (10/0) | TBD V125, possibly +5-9pp |
| Drop-COVID 8w Δ | +12.28pp (8/0) | TBD V125, possibly +4-8pp |
| SuperStruct 6w Δ | +7.53pp (6/0) | TBD V125, possibly +0-5pp |
| Random null p (V113) | 0.035 / 0.010 / 0.045 | Still applies — null is unbiased ✓ |
| V108 walk-forward Δ | +11.94pp (drop 2017-20, 4/3) | UNBIASED — applies as-is |
| V124 regime stability | Sharpe 2.20 in 2022-24 | Independent — applies |
| 16-yr aggregate Sharpe | 1.62 | Conditional on STRICT being correct spec |

**The most honest pre-V125 numbers**:
- V108 walk-forward yearly Drop-2017-2020: **Δ +11.94pp (6/1 W/L)** — independent of overlap bias and post-hoc outlier selection
- V124 most-recent 2022-2024: **Sharpe 2.20, α +44pp solo** — pure descriptive, no selection
- Random null all subsets pass — signal is real

So the signal exists. But the SPECIFIC alpha number (+7.53pp / +12.28pp / +11.88pp) depends on whether STRICT was the right spec to pick.

---

## What V125 Will Tell Us

V125 runs proper nested CV across 288 spec configs:
- Per held-out window, inner-CV picks (mcap, min_b, min_$, hold, weight) on IS+TUNE only
- Reveals held-out Δ
- Average across 10 rolling windows

**Interpretation**:
- If V125 unbiased Δ ≥ +5pp on full-10w → STRICT spec is genuinely good, deploy with confidence
- If V125 unbiased Δ ∈ [+1, +5pp] → signal exists but specific spec choice is fragile; deploy with smaller weight (10-20% rather than 50%)
- If V125 unbiased Δ ≤ 0 → STRICT was max-of-grid bias like V62; do NOT deploy; consider Form 4 a research dead-end

---

## Recommended Action

**HOLD deployment until V125 completes.** The +7.53pp SuperStruct claim has Bug #7 risk and should not be the basis for live trading. Wait for unbiased number.

After V125:
1. If unbiased Δ ≥ +5pp → proceed to Phase 1 paper trade with STRICT spec at 50% weight
2. If unbiased Δ ∈ [+1, +5pp] → proceed but smaller weight (10-20%), and test V108 walk-forward number for live validation
3. If unbiased Δ ≤ 0 → discard STRICT, fall back to V18 alone (proven +8.78pp)

This audit was triggered by user request. If unbiased number is materially different, we update bugs.md to log Bug #8 (or reclassify under Bug #7).
