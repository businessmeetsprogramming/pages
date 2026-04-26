---
layout: default
title: 13F Follower — Final Summary
---

# 13F Follower — Final Project Summary

**Date**: 2026-04-26
**Status**: Phase 1 (13F-only data) closed. Strategy locked: **V18 50/50 (V13 + V17b-1)**.
**Headline (post Bug #7 nested CV)**: Avg unbiased held-out α = **+8.78pp/yr** (V18 fixed config = +9.15pp; shrinkage -0.37pp ✓ V18 passes Bug #7 audit). Bootstrap 95% CI on OOS Sharpe **[0.30, 2.58]**, Full OOS Sharpe **1.26**.

---

## TL;DR

After **84 versions**, **7 confirmed bugs**, and multiple independent audits, the deployable 13F-follower strategy converges back to the original V18 50/50 ensemble — exactly what we had on day one of the research. Every "breakthrough" (V21B4 mid-cap, V55 small-cap, V62 specialist, V80 Dirichlet ensemble) turned out to be a bug-induced mirage:

- V21B4 ← **Bug #4** (stock split adjustment incomplete)
- V59 / V70 ← **Bug #5** (rolling-window single-year outlier amplification)
- V80 multi-variant ensemble ← **Bug #6** (blind ensemble over correlated dud variants)
- **V62 specialist ← Bug #7 (hyperparameter grid search on held-out average)** — discovered today

The 13F-only data ceiling is real. To go higher, we need new data sources (parsed 13D, parsed Form 4 buys-only, N-PORT monthly, IBES revisions). The "small-cap edge" we hunted does not exist in clean, properly-validated 13F data.

---

## 1. Final Conclusion

### 1.1 The deployable strategy: V18 = V13 (50%) + V17b-1 (50%)

| Sleeve | Spec |
|---|---|
| **V13** (50%) — mega-cap mega-fund consensus | top-7 funds by 4y InfoRatio × top-3 holdings (USD MV) × ≥2 consensus × SPY fallback × quarterly rebal |
| **V17b-1** (50%) — high-conviction filter | top-10 funds by 4y InfoRatio × top-5 holdings × **port_wt ≥ 8%** filter × ≥2 consensus × SPY fallback × quarterly rebal |
| **Combine** | weight_S(Q) = 0.5 × V13_weight_S(Q) + 0.5 × V17b_weight_S(Q) |

Each quarter, ranks all 13F filers (≥12 quarters of history) by trailing 4-year InfoRatio (excess return / vol vs SPY on quarterly returns), takes the top-N, filters their published holdings, and constructs a consensus-weighted portfolio. Buy R+45d, sell R+135d (next R+45d), then repeat.

### 1.2 Performance under proper held-out methodology

| Metric | V18 | Note |
|---|---|---|
| **V18 fixed config (current spec)** held-out α | **+9.15pp/yr** | grid-tuned, biased |
| **V18 nested CV (V85, post Bug #7 audit)** held-out α | **+8.78pp/yr** ✓ | unbiased; Δ -0.37pp shrinkage from fixed |
| **Forward expectation** | **+8.78pp/yr** | use this for kill-criterion calibration |
| Held-out 2024-2025 single window Sharpe | 1.44 | |
| Full OOS 2021-2025 Sharpe | 1.26 | |
| 16-year IS+OOS Sharpe | 0.91 | calibrate forward Sharpe to ~0.9 |
| Bootstrap 95% CI on Full OOS Sharpe | [0.30, 2.58] | lower bound > 0 ✓ |
| Worst single year | -8.7pp (V13 alone 2014) | acceptable |
| V13 random null | p < 0.01 | ✓ |
| V13/V17b sleeve correlation | 0.65 | moderate diversification |
| **V18 nested CV W/T/L vs V18 fixed** | 3/0/7 | fixed config slightly outperforms in 7/10 windows but average Δ is small |

### 1.3 Bug #7 — the final one (2026-04-26)

**Bug name**: Hyperparameter grid search on held-out average.

**What happened**: We ran a 96-config grid for V62 (small-cap specialist sleeve): 4 mcap bands × 4 top_n × 3 cons_min × 2 rebal_freq. Each config evaluated under V57 rolling held-out framework. We picked the winning config (V62a = Small $300M-$2B / 15 / 3 / rebal=2) by `df.nlargest(10, 'improvement')` — i.e. **maximum held-out improvement across the 96 configs**.

This is the same family of bug as #2 (V26 OOS-peek) and #6 (correlated dud variants), at the **hyperparameter** layer rather than the weight layer. The +4.38pp/yr improvement claim was max-of-96 selection bias.

**Verification (V83 nested CV)**: Per-window inner grid select on IS+TUNE only; reveal held-out alpha. Result:

| | Avg held-out α | vs V18 alone |
|---|---|---|
| V18 alone | +9.15pp | — |
| V18 + V62a (biased grid claim) | +13.54pp | +4.38pp ❌ biased |
| **V18 + V62 nested CV (unbiased)** | **+4.83pp** | **-4.33pp** ❌ LOSES |

Nested CV picks **8 different V62 configs across 10 windows** — IS+TUNE Sharpe has near-zero power to identify the right V62 hyperparameters ex-ante.

**Random null (V84, 500 trials)**: At fixed V62a config, IR ranking beats random fund selection p=0.018. So V62a config has real signal — but you can't pick it ex-ante. **V62 sleeve is therefore not deployable.**

**V18 own nested CV check (V85)**: same Bug #7 audit applied to V13 (32-config grid) × V17b-1 (72-config grid) × 5 mix weights. Per held-out window, inner-grid select on IS+TUNE Sharpe only.

| | Avg held-out α | shrinkage |
|---|---|---|
| V18 fixed config | +9.15pp | — |
| V18 nested CV (unbiased) | **+8.78pp** | **-0.37pp** ✓ |
| V62 nested CV (comparison) | +4.83pp | -8.71pp ❌ |

V18 shrinks 0.37pp; V62 shrinks 8.71pp. **The 23x ratio is the difference between structural alpha and selection bias.** V18 PASSES Bug #7 audit and is genuinely deployable.

### 1.4 The 7-bug ledger

| # | Bug | Severity | Impact |
|---|---|---|---|
| 1 | `stock_ret_filled = 0` for delistings | MAJOR | overstated small/mid-cap returns by 0.4-1.0pp/yr |
| 2 | V26 OOS-peek weight selection | MAJOR | "Sharpe 2.27" → real held-out 1.48 |
| 3 | V21B4 SPY-fallback alpha inflation | MAJOR | overstated alpha by 0.5-1.5pp |
| 4 | Stock split adjustment incomplete | CRITICAL | **killed V21B4 family entirely**; +12pp → -9pp post-fix |
| 5 | Rolling overlapping windows amplify single-year outliers | MAJOR | V59 micro / V70 first-appearance ruled out |
| 6 | Blind ensemble over correlated dud variants | MAJOR | V80 100k Dirichlet selects worst V62 variant |
| 7 | **Hyperparameter grid search on held-out** | **CRITICAL** | **killed V62 family entirely**; +4.38pp → -4.33pp post-fix |

The "+Sharpe 2.0" target was never real — every breakthrough that broke the 1.5 ceiling was a bug.

### 1.5 What was eliminated and why

| Direction | Verdict | Reason |
|---|---|---|
| V21A1 / V21B4 small-fund mid-cap | DEAD | Bug #4 |
| V46 / V50 13D activist overlay | DEAD | 0% overlap with mega-cap V18 |
| V49 Form 4 raw activity | DEAD | mixes buys+sells, pure overfit |
| V51 quality overlay | DEAD | V18 already filters quality |
| V52 sector specialist overlay | DEAD | single-window 2024-2025 artifact |
| V55 mid $1B-$10B small-cap follow | DEAD | superseded by V62, then V62 itself dies |
| V59 micro <$300M | DEAD | Bug #5 (2020 outlier) |
| V60 sector overlay rolling | DEAD | rolling test 0.00pp |
| **V62 specialist sleeve** | **DEAD** | **Bug #7 (this audit)** |
| V67 emerging fund universe | DEAD | Berk-Green hypothesis fails |
| V70 first-appearance | DEAD | Bug #5 |
| V72 decorrelated consensus | DEAD | rolling test |
| V74 position delta | DEAD | rolling test |
| V75 wolf-pack 13D | DEAD | rolling test |
| V78 persistent consensus | DEAD | rolling test |
| V80 100k Dirichlet ensemble | DEAD | Bug #6 |

**17 research directions eliminated. Only V18 50/50 survives.**

---

## 2. Deployment Plan

### Phase 0 — Pre-launch (week of 2026-05-12)
- ✅ V85 nested CV on V18 — DONE 2026-04-26 — V18 passes Bug #7 audit at -0.37pp shrinkage
- Generate next-quarter positions for R = 2025-12-31 (or 2025-09-30 if not fully filed)
- Lock strategy spec; verify positions look reasonable (mega-cap concentrated, no Carvana-style lottery)

### Phase 1 — Paper trade (2026-05-15 → 2026-08-15)
- 100% paper, real prices
- Compare actual fills to backtest (slippage, missed)
- Verify total-return computation matches CRSP cum_tr

### Phase 2 — Small allocation (2026-08-15+, 5% portfolio)
- Quarterly rebalance on R+45d
- **Kill criterion**: 2 consecutive quarters with α < -3pp vs SPY → review/halt
- **Pause criterion**: 1 quarter with α < -8pp → review (not auto-halt)

### Phase 3 — Target allocation (after 12+ months consistent tracking)
- Scale to 10-15% if Phase 2 tracks within ±3pp of backtest
- Annual nested CV refresh

### Forward expectation calibration

The held-out 2024-2025 Sharpe of 1.44 is the WIDE single-window estimate. **The 16-year IS+OOS Sharpe of 0.91 is the forward expectation.** If realized Sharpe ≈ 0.9 / +6pp alpha, that's "as expected" — not a kill signal. The bootstrap CI [0.30, 2.58] is statistically wide, reflecting genuine uncertainty.

---

## 3. Next Steps

### 3.1 ~~Immediate: V85 — V18 nested CV~~ DONE (2026-04-26)
V85 ran nested CV on the V13 32-config × V17b-1 72-config × 5-weight grid. Result: V18 unbiased held-out α = **+8.78pp** (vs V18 fixed +9.15pp; shrinkage -0.37pp). V18 PASSES Bug #7 audit. Forward expectation locked at **+8.78pp/yr / ~0.9 Sharpe**.

### 3.2 Phase 2 research direction (post-deploy, parallel)

13F-only is closed. To unlock incremental alpha, open new data sources. Priority order:

| Data source | Expected α lift | Effort | Difficulty |
|---|---|---|---|
| **Form 4 buys-only parser** | +2-5pp | 2-3 weeks | medium |
| 13D parsed CUSIP (V43 already indexed 2974 filings) | +1-3pp | 1-2 weeks | medium |
| N-PORT monthly disclosure | +2-4pp | 3-4 weeks | high |
| IBES earnings revisions | +1-2pp | 1 week | low |

**Recommended start**: Form 4 buys-only parser. Insider buys are the strongest single signal in the literature (Lakonishok-Lee 2001, Cohen-Malloy-Pomorski 2012). 9.4M Form 4 filings already indexed; only the parser is missing.

### 3.3 Why "small-cap" is hard from 13F (lesson learned)

The instinct to find a small-cap edge was sound — but every attempt under 13F-only data failed:

| Failure mode | Mechanism |
|---|---|
| **Universe too noisy** | small-cap 13F holdings are rare, distress-prone, prone to reverse splits / delistings — Bug #4 / Bug #1 hit small-cap hardest |
| **Selection bias amplified** | small-cap returns are fat-tailed, so any grid-search-on-held-out (Bug #5/#6/#7) finds spurious "winners" much more easily |
| **No fund-skill signal in 13F** | IR-ranking at fixed V62a config has p=0.018 (real), but you can't identify V62a from IS+TUNE alone (V83 finding). Specialist funds' edge is too thin to survive the data noise |
| **Quarter-end snapshot** | small-cap fund flows during the quarter are invisible; we only see the residual at quarter-end |
| **45-day reporting lag** | small-cap catalysts are often priced-in within 45d; following at R+45 is too late |

**Conclusion: small-cap follower strategies are not viable from 13F-only data.** This isn't a failure of the small-cap thesis — it's a data limitation. Form 4 insider buys-only and 13D activist filings have better signal-to-noise on small-cap names because those data sources reflect direct conviction (insider buying their own stock; activist taking 5%+ stake) rather than residual quarter-end positioning.

Academic precedents: Cohen-Malloy-Pomorski (2012) finds insider buys have 2-3× the alpha in small-caps vs large-caps; Coffee (2022) and Boyson-Mooradian (2023) find wolf-pack 13D (2+ activists ≤90d) preserves alpha post-2010 while single-activist 13D has decayed. Our V75 wolf-pack 13D attempt was promising but failed under rolling test — possibly a hyperparameter issue (Bug #7 risk), worth revisiting with proper nested CV after the Form 4 parser is built.

### 3.4 Process improvements going forward

The Bug #2 / Bug #6 / Bug #7 family is the same disease at three layers: **maximize anything on held-out data → upward bias**. Going forward:

1. **Hyperparameter selection**: nested CV per held-out window, never global.
2. **Weight selection in ensembles**: blind selection on IS+TUNE Sharpe only.
3. **Strategy selection across variants**: blind selection on IS+TUNE Sharpe only; treat held purely as reveal.
4. **Random null on the FINAL deployable spec**: not just on individual sleeves.
5. **Annual nested CV refresh** after deployment.

---

## 4. File Inventory

| File | Purpose |
|---|---|
| `DEPLOYMENT_BLUEPRINT.md` | Frozen strategy spec |
| `bugs.md` | 7-bug log with RCA + lessons |
| `13F_FOLLOWER_REPORT.md` | Research narrative (historical, includes superseded V62 finding) |
| `scripts/v44_rerun_v3.py` | V18 quarterly returns reproduction |
| `scripts/v58_live_position.py` | Live position generator |
| `scripts/v83_v62_nested_cv.py` | Bug #7 verification (proves V62 not deployable) |
| `scripts/v84_v62_random_null.py` | V62 IR-signal test (500 trials) |
| `data/cache/v13_panels_v3.pkl` | Final split-adjusted, delisting-fixed panel |
| `data/cache/v44_v3_quarterly.parquet` | Final returns (V13/V17b/V18/SPY) |

---

## 5. One-paragraph deployment summary

After 7 confirmed bugs and an independent audit, the deployable 13F-follower strategy is the **V18 50/50 ensemble of V13 (top-7 funds, top-3 holdings, ≥2 consensus, SPY fallback) and V17b-1 (top-10 funds, top-5 holdings, ≥8% port-wt conviction, ≥2 consensus, SPY fallback)** with quarterly rebalance on R+45d. Average rolling 10-window held-out α = **+9.15pp/yr**; bootstrap 95% CI on full-OOS Sharpe **[0.30, 2.58]**. Forward expectation calibrated to ~0.9 Sharpe / +6pp alpha. The V62 small-cap specialist sleeve was eliminated when nested CV (V83) revealed the +4.38pp improvement was hyperparameter selection bias (Bug #7); under proper per-window inner grid selection, V62 LOSES to V18 alone by 4.33pp/yr. The V62 IR-ranking has signal at the fixed V62a config (V84 p=0.018), but the right config is not identifiable ex-ante from IS+TUNE Sharpe — so V62 is not deployable in its current form. **Deploy V18 alone, paper-trade for one quarter, then 5% portfolio with kill criterion at 2 consecutive sub-(-3pp) quarters.** Phase 2 research focuses on parsed Form 4 insider buys-only as the most promising path beyond the 13F-only ceiling.

---

## 中文摘要

**最终结论**：经过 84 个版本和 7 个 bug 的修复，可部署策略最终回归到最初的 V18 50/50 双 sleeve（V13 mega-cap 共识 + V17b-1 高 conviction 过滤）。年化 alpha **+9.15pp**，Bootstrap 95% CI Sharpe **[0.30, 2.58]**。

**Bug #7（今日发现）**：V62 small-cap specialist 看似 +4.38pp 的改进，是 96 个超参数组合在 held-out 上选优的偏差。Nested CV 结果是 -4.33pp（输给 V18 alone）。这是 Bug #2 / #6 / #7 同一族 bug 在不同层面的表现：**"在 held-out 上最大化任何东西"**。

**小盘股结论**：13F-only 数据下做不出小盘股策略。所有尝试（V21B4、V55、V59、V62 等）都被各种 bug 杀掉。要做小盘股需要新数据源（Form 4 内幕买入解析、13D activist），而不是 13F。

**下一步**：
1. **立即**：跑 V85（V18 nested CV）确认 V18 自己没踩 Bug #7（预期收缩 1-3pp）
2. **Phase 1**：纸面交易一个季度
3. **Phase 2**：5% 仓位实盘
4. **Phase 3 研究方向**：Form 4 buys-only 解析器（最强单一信号，9.4M filings 已索引，只缺解析器）
