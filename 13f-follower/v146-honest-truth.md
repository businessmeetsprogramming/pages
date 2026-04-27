# V146 UNIFIED Grid Search — Honest Truth

**Date**: 2026-04-27 PM (final audit)
**Status**: V140's "PERFECT 10/0 W/L" was sequential-selection bias. Definitive Bug #7 unified test reveals true alpha.

## Headline

V140 was **path-dependent**: testing STRICT (fail) → V136 add Form 144 (pass) → V140 add 10b5-1 (perfect) used hindsight on held-out at each step. V146 tests all 972 filter combinations simultaneously per-window — the proper Bug #7-clean unified test.

| Subset | V140 sequential claim | **V146 unified unbiased** | Shrinkage |
|---|---|---|---|
| Full 10w | +13.58pp (10/0) | **+4.83pp (7/3)** | -8.75pp |
| Drop-COVID 8w | +12.55pp (8/0) | **+3.58pp (6/2)** | -8.97pp |
| SuperStruct 6w | +10.07pp (6/0) | +5.85pp (5/1) | -4.22pp |
| **95% CI Full** | [+8.63, +18.32] excludes 0 | **[-3.00, +12.40] INCLUDES 0** ❌ |

**95% CI includes zero — NOT statistically significant after unified grid correction.**

## Pattern Recognition

This is the **3rd time** we've seen this:
1. V62 specialist: claimed +4.38pp → unbiased -4.33pp (8.71pp shrinkage)
2. STRICT (V125 audit): claimed +11.88pp → unbiased +3.83pp (8.05pp shrinkage)
3. **V140 (V146 audit): claimed +13.58pp → unbiased +4.83pp (8.75pp shrinkage)**

Same magnitude shrinkage each time → confirms the systematic Bug #7 mechanism.

## What V146 Did Reveal

Selected configs across 10 windows show common pattern:
- **positive_ROA quality filter**: in 9/10 windows ✓
- **micro <$300M or $100M-$300M mcap**: in all 10 ✓
- **180d hold period**: in 8/10 ✓
- **Form 144 / 10b5-1 filters**: scattered (some windows yes, some no)

So the real alpha (~+5pp Full) lives at:
- mcap $100M-$300M (Goldilocks zone)
- ≥3 unique insiders Form 4 P-buy
- positive ROA (quality filter)
- 180-day hold
- Form 144 / 10b5-1 filters NOT essential

## Honest Forward Expectation

| Strategy | Sharpe | α SPY | Status |
|---|---|---|---|
| **V18 alone** | **1.16** | **+8.78pp** | ✅ Verified clean (V85 nested CV) |
| ~~V18 + V140 sequential claim~~ | ~~1.46~~ | ~~+16.5pp~~ | ❌ Partial Bug #7 |
| **V18 + V146 unified unbiased** | **~1.20** | **~+13pp** (V18 +8 + V146 lift +5) | ⚠ CI includes 0, marginal |

**V18 alone remains the only DEPLOYABLE Phase 2-clean strategy.**

V146 unified suggests there's *some* Phase 2 lift (+5pp super-struct) but not statistically significant. Could deploy at **small weight (10-20%)** with caveat that effect may not materialize.

## Lesson

Phase 2 alt-data search **definitively complete after 11 hypothesis × unified grid audit**. Form 4 + Form 144 + 10b5-1 + Quality combination shows weak structural alpha but cannot reject null.

For deployment: **stick with V18 alone**. Optionally add small experimental V146-spec sleeve (~10-20% weight) to test live with low risk.
