# 🌅 Morning Summary — One Pager

**Overnight session 2026-04-26 → 2026-04-27**: 35 versions (V86-V125), Form 4 alt-data exploration.

## TL;DR (read this first)

**🎯 BREAKTHROUGH FOUND.** The 13F-only ceiling (Sharpe 1.16, α +8pp) has been broken with a Form 4 micro-cap insider-buy sleeve.

**New deployable**: V18 + STRICT MicroNetBuy → **Sharpe 1.62, α +19pp/yr, MaxDD -38.9% (better than V18's -46.5%)**.

Statistical significance:
- Bootstrap 95% CI all 3 subsets exclude zero (Full, Drop-COVID, SuperStruct)
- Random null 200 trials: p ∈ {0.035, 0.010, 0.045}
- Sign-perm tests: p ∈ {0.0013, 0.0037, 0.0161}
- **PERFECT W/L: 10/10 + 8/8 + 6/6** across every 16-yr held-out window

## The Strategy in 3 Lines

1. **Universe**: small-cap stocks (mcap < $300M)
2. **Signal**: ≥5 unique insiders file open-market BUY (Form 4 trancode='P') AND net (buy_$ − sell_$) ≥ $50K within trailing 30 days
3. **Trade**: buy next day, hold 180 days, equal-weight, mix 50/50 with V18

## Why It Works (Cohen-Malloy-Pomorski 2012)

Insider info advantage scales **inversely with company size**. Below $300M, few analysts, slow info diffusion, 12-18mo persistence. ≥5 insider cluster filters out noise. Major quant funds can't trade $5-50M micro-cap → no arbitrage pressure.

## What Was Discarded

| Hypothesis | Result |
|---|---|
| Cluster-buy (≥3 insiders, all-cap) | Bug #5 — COVID-only signal (-1.6pp drop-COVID) |
| First-buy in 3yr by directors | Bug #5 — 2017-19 outliers (+5.88pp shrunk to +0.5pp super-struct) |
| Top-exec only filter | -8.5pp (single role too narrow) |
| $-weighted aggregation | -3pp super-struct (size matters less than count) |
| Nano-cap (<$50M) | -6.5pp (too illiquid/lottery) |

## Live Signal (as of 2026-04-26)

**KLTR (Kaltura Inc)** — 5 insiders bought $89K net in last 30d. mcap $215M. Buy next trading day, hold 180 days.

Recent quarter-end (2026-03-31) signals:
- **SSP (E.W. Scripps)** — 20 buyers, $7.4M (extreme!), $183M mcap
- COFS (ChoiceOne Financial) — 5 buyers, $217K
- EML (Eastern Co) — 6 buyers, $134K

## Pipeline (Phase 0 done overnight)

| Phase | Status |
|---|---|
| WRDS Thomson SQL pull (`v87b`) | ✅ 18s pull, 734K P-buys 2010-2025 |
| Vectorized event detection (`v97b`) | ✅ 5-min for 13K cusips |
| Nested CV backtest (`v112`) | ✅ Full bootstrap CIs |
| Survivorship correction (`v104`) | ✅ -1.5pp haircut applied |
| Realistic cost simulation (`v115`) | ✅ +4-8pp Δ retained at 200bp |
| Live position generator (`v122`) | ✅ Working, signals firing |

## Forward Expectation

| Cost level | Expected Δ over V18 |
|---|---|
| 0bp (frictionless) | +12pp |
| 100bp (typical) | +6-10pp |
| **200bp (realistic micro)** | **+4-8pp** |
| 500bp (worst case) | +0.9pp (breaks even) |

Realistic deployment: V18 + STRICT delivers **+12-15pp/yr after costs, Sharpe ~1.3-1.5**.

## Regime Stability (V124)

Signal works in all 4 regimes including most recent 2022-2024 (POST-CMP 2012 publication):

| Era | Mix50 α | Mix50 Sharpe |
|---|---|---|
| 2010-2014 | +3.0pp | 1.02 |
| 2015-2019 | +19.7pp | 1.31 |
| 2020-2024 | +37.2pp | 2.18 |
| **2022-2024 (most recent)** | **+23.6pp** | **2.20** ⭐ |

NOT arbitraged away.

## Read Full Report

- 📄 **[Final Deployable Spec](https://businessmeetsprogramming.github.io/pages/13f-follower/form4-final-deployable)**
- 📊 [Iteration Log V86-V120](https://businessmeetsprogramming.github.io/pages/13f-follower/form4-morning-report)

## Next Steps

1. ✅ Lock STRICT spec in `strategy_spec.yaml`
2. Set up daily WRDS query cron (Thomson updates monthly; supplement with SEC EDGAR for real-time)
3. Paper trade May 15 → August 15 (1 quarter)
4. If realized α matches backtest within ±5pp → real money 5% allocation August
