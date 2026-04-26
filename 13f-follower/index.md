---
layout: default
title: 13F Follower Research
---

# 13F Follower Research

A long-running quantitative project: can we generate alpha by following the holdings disclosed in SEC Form 13F by skilled hedge funds? After 84 strategy versions, 7 confirmed bugs, and multiple independent audits, the answer is **yes — but only with mega-cap consensus, and only at +9.15pp/yr**.

## Final Status (2026-04-26)

**Deployable strategy**: V18 50/50 = V13 (mega-cap mega-fund consensus) + V17b-1 (high-conviction filter).

**Performance** (rolling 10-window held-out, V57 framework):
- Avg held-out α: **+9.15pp/yr**
- Held-out 2024-2025 single-window Sharpe: 1.44
- Bootstrap 95% CI on Full OOS Sharpe: [0.30, 2.58]
- Forward expectation: ~0.9 Sharpe / +6pp alpha (calibrated to 16-yr IS+OOS)

## Reports

- [Final Project Summary (post Bug #7 audit)](final-summary) — comprehensive deployable spec, 7-bug ledger, 17 eliminated research directions, Phase 2 roadmap (2026-04-26)

## Key Lessons

1. **Bug #2 / #6 / #7 are the same disease**: maximizing any metric on held-out data biases winners upward. Audit at the weight, ensemble, AND hyperparameter layers.
2. **Stock split adjustment is critical** for small/mid-cap backtests. Bug #4 (incomplete adjustment) killed an entire research family (V21B4) that had looked like a +12pp/yr breakthrough.
3. **Small-cap follower strategies do not survive proper validation** under 13F-only data. Universe is too noisy, fat-tailed, and prone to delisting / reverse-split artifacts.
4. **Forward expectation should calibrate to 16-yr IS Sharpe (~0.9)**, not the optimistic held-out 1.44. Bootstrap CI [0.30, 2.58] reflects genuine uncertainty.

## Phase 2 Direction

13F-only data ceiling is real. Next: parsed Form 4 insider buys-only (9.4M filings already indexed). Insider buys are the strongest single signal in the literature (Lakonishok-Lee 2001, Cohen-Malloy-Pomorski 2012). Expected lift +2-5pp.
