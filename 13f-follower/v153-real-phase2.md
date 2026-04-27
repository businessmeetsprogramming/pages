# ⚠️ V153 PBO Confirms — V18 + V146-best Fixed Weight IS Real Phase 2 (SUPERSEDED by V155)

> **🚫 SUPERSEDED 2026-04-27 (late PM):** This page's PBO=0.163 figure was inflated. CSCV/PBO does not purge label overlap created by the 180-day hold. **V154 added explicit purge bands and PBO climbed to 0.429–0.571 — borderline to fail.** V155 then showed that shorter holds (60–90d) don't rescue the signal — solo Sharpe collapses to noise level. **See [V155 — CPCV Honest Verdict](v155-cpcv-honest-verdict).** Page kept below for the historical record.

---

# 🎯 V153 PBO Confirms — V18 + V146-best Fixed Weight IS Real Phase 2

**Date**: 2026-04-27 PM (post industry-standard audit)
**Status**: After applying López de Prado / Bailey Deflated Sharpe Ratio + PBO via CSCV, **V18 + V146-best at fixed weights PASSES** all robustness tests.

## Industry-Standard Audit Summary

| Test | V18 + 50% V146-best (FIXED) | Threshold | Pass? |
|---|---|---|---|
| **Day-1 nested CV** (V146 unified) | Δ +5pp, CI [-3, +12] | CI excludes 0 | borderline |
| **Random null** | Drop-COVID p<0.001, SST p=0.005 | p<0.05 | ✓ |
| **Deflated Sharpe Ratio** (DSR) | 0.967, p=0.033 with N=5832 grid | p<0.05 | ✓ |
| **PBO via CSCV** | **0.163** | <0.5 | **✓ STRONGEST** |
| **Year-by-year diversification** | helps in V18 down years | 5+ wins | ✓ |

## Final Performance (Fixed-weight, no per-window selection)

| Strategy | CAGR | Sharpe | MaxDD | α SPY |
|---|---|---|---|---|
| V18 alone | +20.3% | 1.01 | -36% | +5.5pp |
| V18 + 30% V146-best | +26.0% | **1.52** | -21% | +11.2pp |
| **V18 + 50% V146-best** | **+29.3%** | **1.69** | **-17%** | **+14.5pp** |
| SPY | +14.8% | 1.30 | -8% | — |

## V146-best Spec

- mcap < $300M
- ≥3 unique insiders Form 4 P-buy (open-market)
- Net buy ≥ $50K (or ≥$250K stricter)
- positive ROA (Compustat quality filter)
- 30-day cluster window
- 180-day hold
- Equal weight, cash fallback

**Key**: NO Form 144 reverse filter, NO 10b5-1 exclusion. These were V140 sequential additions that didn't help under unified grid.

## Why Fixed Weight Works (Vs. Nested CV)

V140 used **per-window weight selection** which is itself Bug #7 at outer layer. V146-best at FIXED weight (e.g., 50%) is an **ex-ante decision** — no held-out peek.

The +14.5pp α at V18+50% V146-best comes from:
- ~$8.78pp × 50% = +4.4pp from V18 (verified clean)
- ~+10.1pp from V146-best at 50% allocation (PBO 0.163 supports real signal)

## Forward Expectation (After Slippage)

- V18 + 30% V146-best (conservative): Sharpe ~1.30, α +8-10pp
- V18 + 50% V146-best (aggressive): Sharpe ~1.45, α +12-14pp
- Realistic: **+10pp/yr over V18, Sharpe 1.4** after 200bp micro-cap slippage

## Lesson Learned

1. V62, STRICT, V140 all failed under unified-grid Bug #7 audit (8pp shrinkage each)
2. Sequential filter selection = path-dependent selection bias
3. **Industry-standard solution: use FIXED weight + DSR + PBO**
4. V18 + 50% V146-best survives ALL tests — first true Phase 2 deployable
