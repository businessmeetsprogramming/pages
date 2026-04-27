---
layout: default
title: 13F Follower Research
---

# 13F Follower Research

A long-running quantitative project: can we generate alpha by following the holdings disclosed in SEC Form 13F by skilled hedge funds? After 85 strategy versions, 7 confirmed bugs, and multiple independent audits including nested cross-validation on every layer, the answer is **yes — but only with mega-cap consensus, and only at ~+8.78pp/yr (unbiased)**.

## Final Status (2026-04-27 late PM — project converged)

**Deployable strategy**: V18 50/50 = V13 (mega-cap mega-fund consensus) + V17b-1 (high-conviction filter). **No Phase 2 small-cap overlay survives leakage-aware testing.** See [V155 verdict](v155-cpcv-honest-verdict) for the full Bailey–López de Prado triple.

**Performance** (rolling 10-window held-out, V57 framework, post Bug #7 nested-CV audit):
- V18 fixed config (current spec): +9.15pp/yr (biased grid claim)
- **V18 nested CV (unbiased forward expectation)**: **+8.78pp/yr** ✓ shrinkage only -0.37pp, passes Bug #7 audit
- Held-out 2024-2025 single-window Sharpe: 1.44
- Bootstrap 95% CI on Full OOS Sharpe: [0.30, 2.58]
- Forward expectation calibration: ~0.9 Sharpe / +6-8pp alpha

**Comparison — V62 small-cap specialist sleeve** (eliminated):
- V62 fixed config: +13.54pp/yr (biased grid claim)
- V62 nested CV: +4.83pp/yr (loses to V18 alone by 4.33pp) ❌

V62 collapses by 8.71pp under nested CV. V18 collapses by only 0.37pp. **That gap is the difference between structural alpha and selection bias.**

## Reports

- 🏁 **[V155 — CPCV Honest Verdict (PROJECT CONVERGED)](v155-cpcv-honest-verdict)** — **FINAL** (2026-04-27 late PM). CPCV with leakage-matched purge reverses V153. V18+V146-best fails at PBO 0.43–0.57. Short-hold rescue collapses to noise. **V18 alone is the deployable answer.**
- ⚠️ [V153 — V18+V146-best PBO 0.163 (SUPERSEDED by V155)](v153-real-phase2) — CSCV figure inflated by label leakage; reversed by CPCV
- ⚠️ [V146 UNIFIED Grid — V140 was Bug #7 partial bias](v146-honest-truth) — V140 claim shrunk 8.75pp under unified grid
- 🏆 [V3 — V18 + V140 (PRE-AUDIT, OVERSTATED)](morning-final-v3) — superseded by V146
- 🏆 [V140 PERFECT W/L (PRE-AUDIT)](v140-perfect-wl) — superseded by V146
- 🚀 [V2 FINAL — V18 + V136 (superseded by V140)](morning-final-v2) — V136 = V140 without 10b5-1 filter
- 🎯 [V136 Form 144 Reverse Filter Breakthrough](v136-form144-breakthrough) — original Form 144 finding
- ⚠️ [V18+STRICT Bug Audit](v18-strict-bug-audit) — STRICT spec failed Bug #7 audit (2026-04-27 AM)
- [🌅 Morning One-Pager TL;DR (V1, pre-audit)](morning-tldr) — superseded
- [Form 4 FINAL Deployable Spec (V1, pre-audit)](form4-final-deployable) — superseded
- [Form 4 Iteration Log V86-V124](form4-morning-report) — overnight research history
- [Phase 1 Final Summary (13F-only)](final-summary) — V18 baseline, 7-bug ledger

## Full audit & validation chain

| Stage | Test | Result |
|---|---|---|
| Stock split adjustment | panel v3 rebuild | clean |
| Delisting return | -10% mean (not 0% fillna) | clean |
| V13 random null | 1000 trials | p < 0.01 ✓ |
| V18 Bug #7 nested CV (V85, hyperparameter audit) | 10 rolling windows, inner grid on IS+TUNE only | **+8.78pp (Δ -0.37pp from fixed)** ✓ |
| V62 Bug #7 nested CV (V83) | same | -4.33pp ❌ killed V62 |
| V62 random null (V84) | 500 trials at fixed V62a config | p=0.018 (signal exists but unidentifiable ex-ante) |
| V18 sleeve correlation (V13/V17b) | 0.65 | moderate diversification ✓ |
| 16-yr IS Sharpe vs OOS Sharpe | 0.91 vs 1.26 | OOS > IS, not regime-fragile ✓ |
| Bootstrap CI lower bound | +0.30 | statistically defensible ✓ |

## Key Lessons

1. **Bug #2 / #6 / #7 are the same disease**: maximizing any metric on held-out data biases winners upward. Audit at the weight, ensemble, AND hyperparameter layers.
2. **Stock split adjustment is critical** for small/mid-cap backtests. Bug #4 (incomplete adjustment) killed an entire research family (V21B4) that had looked like a +12pp/yr breakthrough.
3. **Small-cap follower strategies do not survive proper validation** under 13F-only data. Universe is too noisy, fat-tailed, and prone to delisting / reverse-split artifacts.
4. **Forward expectation should calibrate to 16-yr IS Sharpe (~0.9)**, not the optimistic held-out 1.44. Bootstrap CI [0.30, 2.58] reflects genuine uncertainty.

## Phase 2 Direction

13F-only data ceiling is real. Next: parsed Form 4 insider buys-only (9.4M filings already indexed). Insider buys are the strongest single signal in the literature (Lakonishok-Lee 2001, Cohen-Malloy-Pomorski 2012). Expected lift +2-5pp.

## Phase 2 Honest Outcome (2026-04-27 final)

After the full Bailey–López de Prado audit stack (DSR + PBO via CSCV + CPCV with purge), **every Phase 2 small-cap overlay candidate failed at least one test**:

- V62 small-cap → killed by Bug #7 nested CV (-8.71pp shrinkage)
- STRICT spec → killed by Bug #7 unified audit
- V140 (Form 144 + 10b5-1 filter) → killed by Bug #7 unified audit (-8.75pp)
- **V146-best** → killed by leakage-aware CPCV (PBO 0.43–0.57)

The pattern is consistent: small-cap insider-buy variants pass coarser tests but fail under tighter validation. **Continuing to iterate on this signal family is unlikely to produce a deployable overlay.**

Future work should pivot to a fundamentally different signal source (factor-style, options-flow, alt-data) rather than continue refining small-cap insider-buy specs.
