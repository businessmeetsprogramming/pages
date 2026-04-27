# ⚠️ V155 — CPCV Reverses V153: V18 + V146-best Fails Leakage-Aware Test

**Date**: 2026-04-27 (late PM, supersedes V153)
**Status**: V153's PBO=0.163 was inflated. With proper time-series purging (CPCV), V18+50% V146-best lands at PBO 0.43–0.57. **V18 alone is the only strategy that survives the full Bailey–López de Prado triple plus random null.**

## TL;DR (honest)

- **V153 was the optimistic figure** — CSCV/PBO doesn't purge label overlap created by the 180-day hold.
- **V154 added explicit purge bands** matching the 2-quarter forward-return window. PBO climbs from 0.143 (no purge) → 0.429 (purge=1) → 0.571 (purge=2, leakage-matched).
- **V155 tested shorter holds** (60d/90d/120d) to escape the leakage problem. Purge-matched CPCV passes, but solo V146-best Sharpe collapses to 0.55–0.65 — statistically indistinguishable from cusip-shuffle noise (0.57–0.63).
- **Net:** the small-cap "real Phase 2" claim does not survive leakage-aware testing. **Ship V18 alone.**

## The full test stack — V18 + 50% V146-best (180d hold)

| Test | Method | Result | Verdict |
|---|---|---|---|
| Bug #7 unified grid (V146) | 972 configs simultaneously | Δ +5pp; 95% CI [-3.00, +12.40] | borderline (CI includes 0) |
| Random null (cusip-shuffle) | Drop-COVID p<0.001, SST p=0.005 | nominal real | PASS |
| Deflated Sharpe (DSR) | López de Prado, N=5832 trials | p = 0.033 | PASS |
| **PBO via CSCV (V153, no purge)** | S=10 distinct strategies | **0.163** | **PASS (but optimistic)** |
| **CPCV purge=1 (V154)** | half of 180d label window | **0.429** | borderline |
| **CPCV purge=2 (V154)** | full 180d label window | **0.571** | **FAIL** |

V153 looked clean because CSCV blocks neighboring quarters into the same train/test partitions, and those neighboring quarters share return information from the 180d forward window. Once we purge the overlap (CPCV), the IS-best winner shifts away from V18+50% V146-best to V18+50% STRICT (9/14 vs 4/14 of properly-purged folds).

## V155 hold sensitivity — can we save it with a shorter hold?

If we shorten the hold to ≤90d, then purge=1 fully covers the 1-quarter leakage and the test is leakage-clean. But the signal disappears:

| Hold | Solo Sharpe | V18+50% Sharpe | CPCV PBO (matched purge) | V18+50% IS-best wins | Note |
|---|---|---|---|---|---|
| 60d  | 0.557 | 1.221 | 0.393 (PASS) | 8/28 (29%) | leakage-clean but signal noise-level |
| 90d  | 0.648 | 1.158 | 0.393 (PASS) | 3/28 (11%) | **SPY wins IS-best 15/28 (54%)** |
| 120d | 0.965 | 1.387 | 0.571 (FAIL) | 4/14 (29%) | partial leakage, fails CPCV |
| 180d | 1.381 | 1.689 | 0.429–0.571 | varies by strategy set | flagship; leakage-inflated |

Solo V146-best at 60–90d (Sharpe 0.55–0.65) is **statistically indistinguishable from random cusip-shuffle nulls** (0.57–0.63). The high 180d Sharpe was the 2-quarter window borrowing return information from the next quarter into the current one.

## Honest deployment hierarchy

1. **V18 alone** — Sharpe 1.01, +8.78pp alpha vs SPY, passes Bug #7 unified audit, passes DSR, passes CPCV. **Deploy as-is.**
2. **V18 + 10–15% V146-best (180d)** — marginal; only worth running if you treat CPCV purge=2 evidence as too conservative. Adds maybe +1–2pp at the price of CPCV ambiguity.
3. ❌ **V18 + 30–50% V146-best (180d)** — looked best in V150/V153. Fails the leakage-aware test. **Reject for live capital.**
4. ❌ **V18 + V146-best at short hold** — fails on signal collapse. Reject.

## Methodological lesson

**CSCV (Bailey 2014) is a multi-testing penalty, not a leakage shield.**

When forward-return windows exceed the validation block size, CSCV under-states overfitting because adjacent test/train blocks share return information. CPCV (López de Prado 2018) with `purge ≥ ceil(hold / block_size)` is the right test for any strategy with hold ≥ block size.

**Going forward, every strategy claim in this repo should report CPCV with leakage-matched purge alongside DSR.** A signal that passes only un-purged CSCV (like V146-best did) is not yet validated.

## What this means for the project

The Phase 2 search is honestly converged.

- **All four "real signal" Phase 2 candidates** (V62 small-cap, STRICT, V140 with Form 144, V146-best) showed the same pattern under proper auditing: looked great in fixed-window grids, lost ~5–10pp under unified Bug #7 audit, and now V146-best loses again under leakage-aware CPCV.
- **The structural finding is robust**: 13F mega-cap consensus (V18) is real; small-cap insider-buy overlays are not, once you control for both selection bias and label leakage.
- **Deploy V18 alone**. Treat any small-cap overlay as experimental at ≤15% weight, with the explicit understanding that CPCV evidence is borderline.

## Files

- `scripts/v150_v18_v146_fixed_weight.py` — fixed-weight backtest
- `scripts/v151_dsr_pbo.py` — Deflated Sharpe Ratio implementation
- `scripts/v152_pbo_cscv.py` — early CSCV (later replaced)
- `scripts/v153_pbo_distinct.py` — CSCV with distinct strategies; PBO 0.163
- `scripts/v154_cpcv_purged.py` — CPCV with explicit purge; reveals PBO 0.43–0.57
- `scripts/v155_short_hold_cpcv.py` — hold sensitivity + leakage-matched purge

## Where this leaves us

The 13F-follower research project has reached its honest end-state:

- **V18 (mega-cap consensus)**: +8.78pp/yr unbiased, Sharpe 1.01 → **deployable**
- **Phase 2 small-cap overlays**: every candidate failed at least one of {unified Bug #7 audit, leakage-aware CPCV, signal-vs-noise on shorter holds}

Time to either ship V18 to production or pivot to a fundamentally different signal family (factor-style, options-flow, alt-data) rather than continue iterating small-cap insider-buy variants that keep failing the same way.
