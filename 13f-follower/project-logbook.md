# 13F-Follower 项目完整 Logbook

**Date**: 2026-04-27 (Phase 2 honestly converged)
**Versions covered**: V0 → V155
**Primary repo**: `100x_research/`
**GitHub Pages**: https://businessmeetsprogramming.github.io/pages/13f-follower/

---

## 0. 一页纸总结

| 维度 | 答案 |
|---|---|
| 起点假设 | "跟随顶尖 hedge fund 的 13F 持仓可以赚钱" |
| 数据基础 | SEC 13F (季频持仓) + Form 4 (insider 实时交易) + WRDS CRSP/Compustat + Form 144/10b5-1 |
| 唯一活下来的策略 | **V18 = V13 (mega-cap 共识) + V17b-1 (高信念过滤) 50/50 混合** |
| V18 真实 alpha | **+8.78pp/yr (unbiased nested CV), Sharpe ~1.0** |
| 全部 Phase 2 small-cap 候选 | 全死了 (V62 / STRICT / V140 / V146-best 都倒在 Bug #7 或 label leakage 上) |
| 找到的 bugs 数量 | 8 个 (Bug #1–#7 + label leakage) |
| 收敛结论 | 13F-only 数据天花板真实存在；继续在 small-cap insider-buy 这个家族上迭代不太可能产出新增信号 |

---

## 1. 项目目标 & 起点假设

### 1.1 我们想回答的问题

> "如果我能拿到顶级对冲基金的季度持仓 (SEC 13F)，能不能通过简单的 rule-based 跟单赚到 alpha？如果可以，最多能拿多少？"

### 1.2 学术 background

- **Cohen-Polk-Silli (2010) "Best Ideas"**: 基金经理的 top conviction 持仓显著 outperform 整个 portfolio
- **Wermers et al.**: hedge fund 13F 持仓在 short-term 有信息含量
- **Lakonishok-Lee (2001), Cohen-Malloy-Pomorski (2012)**: insider 买入 (Form 4 P-buy) 是公开数据中最强的单一信号
- 普遍观察：simpler rules (top-K + consensus) 比复杂 ML 更鲁棒

### 1.3 数据资源 (用户提供)

- WRDS 完整接入 (CRSP / Compustat / Thomson Insider Filings / Form 144 / Rule 10b5-1)
- SEC EDGAR 13D bulk 305K 文件
- Yahoo Finance 免费 fallback
- 18 年 (2008–2026) 完整 panel

---

## 2. 主线时间轴：V0 → V155

下面按"重要 milestone"组织，不是逐版本流水账。

### 2.1 V1–V18：13F-only baseline 找到唯一活下来的信号

| 版本 | 核心想法 | 结果 |
|---|---|---|
| V1–V12 | top-N fund × top-K holding 的各种排列组合 | 多数 Sharpe ~0.5–0.8，被多重测试惩罚后归零 |
| **V13** | top-7 funds (按 InfoRatio rank) × top-3 holdings × ≥2 fund consensus, 季频 fund rebal | Sharpe 1.44 OOS, +9pp alpha |
| **V17b-1** | top-10 funds × top-5 holdings × portfolio-weight ≥8% × ≥2 consensus | Sharpe 1.11，独立信号 |
| **V18** | V13 + V17b-1 各 50% (correlation 0.65) | **Sharpe ~1.2 OOS, +8.78pp alpha** |

**V18 的关键性质**:
- mega-cap heavy (避开小盘 noise & delisting trap)
- consensus ≥2 (减少 single-fund idiosyncratic risk)
- top-K holdings (利用 best-ideas 效应)
- 15 个 fund 是 InfoRatio 排序后选出，**这个选 fund 步骤是 in-sample only**

**V18 通过的 audit**:
- Bug #7 unified hyperparameter grid (V85): Δ shrinkage 仅 -0.37pp
- 16-yr IS Sharpe 0.91 vs 10-window OOS Sharpe 1.26 (OOS > IS, 无 regime fragility)
- Bootstrap 95% CI lower bound +0.30 (统计 defensible)
- 后来在 V154 leakage-aware CPCV 下也通过

### 2.2 V19–V90：small-cap "specialist sleeve" 全军覆没

主要思路：试图找一个 **小盘股 sleeve**，可以加在 V18 上面 boost alpha。

| 版本 | 想法 | 死法 |
|---|---|---|
| V21B4 | small-fund × mid-cap | Bug #4 (split adjustment) — 修了之后 +12pp 变成 +0pp |
| V26 | mega-ensemble grid | Bug #2 (OOS-peek filter) — 报 Sharpe 2.27 实际 1.48 |
| V62 | small-cap specialist (top-7 small funds) | Bug #7 — 报 +13pp，nested CV 后只剩 +4.83pp |
| V70 | first-appearance filter | Bug #5 (rolling-window outlier) |
| V80 | blind ensemble | Bug #6 (correlated dud variants) |

**V62 的故事是核心教训**：单窗看是个干净的 small-cap signal (multiple positive years 2013/2018/2020/2022)，不是 Bug #5 的 single-year outlier。但是 V62a 那个 winner 是 from a hyperparameter grid search，**max-of-grid selection bias 让 +13pp 缩到 +4.83pp**。这就是 Bug #7。

### 2.3 V90–V125：转向 Form 4 insider buys

意识到 13F 只是季度快照 (45 天 lag)，开始用 **Form 4 实时 insider buy** 作为 alt-data。

| 版本 | 想法 | 结果 |
|---|---|---|
| V97/V97b | Form 4 P-trans clustering | 工具基础设施 |
| V104 | Quarterly returns with imputation | 处理 sell-date 缺失 |
| V125 | STRICT spec: micro<$300M + ≥3 insiders + $50K + 30d window + 180d hold | 单窗看好，**Bug #7 nested CV 缩水 ~8pp** |

V125 让我们意识到：**Form 4 cluster signal 是真的，但太多 hyperparameter 自由度容易 selection bias**。

### 2.4 V126–V145：alt-data 武器库扩张

| 版本 | 数据源 | 用途 |
|---|---|---|
| V126/V127 | Compustat PEAD (post-earnings drift) | 测试有没有 PEAD-style 信号 — 没出色 |
| V128 | SEC 13D bulk 305K filings | activist filer 识别 |
| V131 | WRDS Form 144 | 内部人申报卖出意图 |
| V132 | Compustat ROA/leverage/CFOA | quality factor |
| V135 | Quality merge 到 Form 4 | Form 4 + ROA 联合 |
| V140 | STRICT + Form 144 reverse + 10b5-1 exclude | **报 +13.58pp, 10/0 win, CI 不含 0** ← 看起来是 home-run |
| V144 | 13D first-time activist | 弱信号 |
| V145 | Form 3 new insider | 没出色 |

V140 看起来太漂亮了，这就是触发 V146 怀疑的原因。

### 2.5 V146：决定性的 Bug #7 unified test

**V146 用 972 个 config 的 unified grid 替代 V125→V136→V140 的 sequential 路径**。

结果：V140 Δ +13.58pp 缩到 V146 unified Δ +5pp，CI [-3, +12] 包含 0。

**V146 的独立结果是它选出了一个 "V146-best" spec**：micro<$300M + ≥3 buyers + $250K + positive ROA + 180d hold (Form 144 filter 和 10b5-1 exclude **都被 reject**)。

详细说明见 [V146_DETAIL.md](V146_DETAIL.md)。

### 2.6 V147–V155：industry-standard 严格审计 V146-best

| 版本 | 测试 | 结果 |
|---|---|---|
| V147 | Lock V146-best, cusip-shuffle null | Drop-COVID p<0.001, SST p=0.005 ✓ |
| V150 | V18 + V146-best fixed-weight backtest | Sharpe 1.69, CAGR +29.3%, α +14.5pp |
| V151 | Deflated Sharpe Ratio (Bailey-López de Prado) | DSR 0.967, p=0.033 ✓ |
| V152/V153 | PBO via CSCV | PBO=0.163 ✓ (但是没 purge label leakage) |
| **V154** | **CPCV with purge gap** | **PBO 0.43–0.57 ⚠ 翻车** |
| V155 | 短 hold (60–90d) 救援尝试 | 信号塌到噪声水平 |

**V155 收尾结论**: V146-best 在 light testing (DSR + random null + un-purged CSCV) 下过，在 heavy testing (CPCV with leakage-matched purge) 下不过。

---

## 3. 8 个 Bugs (项目教训)

| # | Bug | 死法 | 第一次撞 | 修复方法 |
|---|---|---|---|---|
| 1 | `fillna(0)` on delisted stocks | 把 -100% 当 0% | V25 (small-cap 策略) | 实际 lookup last price，缺失就标 -100% |
| 2 | Ensemble OOS-peek 权重选择 | filter `oos_sh ≥ 1.5` 后 rank by joint | V26 | IS + TUNING / HELD-OUT 三段切分 |
| 3 | SPY-fallback inflates alpha | 当策略空仓时 fillin SPY return 而不是 cash | V21B4 | 空仓填 0 (cash) |
| 4 | Stock split adjustment 不全 | small-cap split 没正确 adjust | V21B4 | Panel rebuild with cum_tr factor |
| 5 | Rolling-window outlier | 重叠窗口里同一个 single-year outlier 重复出现 | V61 | 报 drop-COVID + SST 多重 robustness |
| 6 | Blind ensemble over correlated variants | 选 dud variants 的 weighted avg | V80 | 用 IS+TUNE only Sharpe 选，不预先 OOS filter |
| 7 | Hyperparameter grid search on held-out average | max-of-grid selection bias | V62 | Per-window nested CV: 内层 grid 只看 IS+TUNE |
| 8 | **Label leakage when hold ≥ block-size** | CSCV 把相邻 quarter 共享的 forward-return 当独立 | **V154** | **CPCV with purge ≥ ceil(hold / block-size)** |

**核心教训：每一层选择都要被审计**

```
Layer A (个股)         → V13 random null    (V44, p<0.01)
Layer B (sleeve 权重)  → IS+TUNE-only mix   (Bug #2 防御)
Layer C (hyperparameter) → unified grid + nested CV (V85/V146)
Layer D (multi-testing) → Deflated Sharpe Ratio (V151)
Layer E (label leakage) → CPCV with purge   (V154)
```

任何层没审计的策略都算"未验证"。

---

## 4. 理论框架：什么算"真实"信号

我们逐渐沉淀出来一套判断标准：

### 4.1 Necessary conditions (任何一个不过就 reject)

1. **Single-stock null**: 个股 random shuffle 1000 trials, p < 0.05
2. **Bug #7 unified nested CV**: per-window 选 hyperparameter 只看 IS+TUNE，shrinkage < 2pp
3. **Random cusip-null on locked spec**: spec 锁死后 cusip-shuffle 200 trials, p < 0.05
4. **Deflated Sharpe Ratio**: DSR > 0.95 (即 multi-testing-adjusted p < 0.05)
5. **CPCV with leakage-matched purge**: PBO < 0.5

### 4.2 Sufficient conditions (强证据)

- Drop-COVID and SST robustness 都通过
- 16-yr IS Sharpe ≥ 10-window OOS Sharpe (无 regime fragility)
- 信号在多个不同 cluster 标准下稳定 (而不是某个特定 hyperparameter 才出)

### 4.3 V18 vs V146-best 的 score card

| 标准 | V18 | V146-best |
|---|---|---|
| Single-stock null | ✓ p<0.01 | n/a (cluster signal) |
| Bug #7 unified nested CV | ✓ Δ -0.37pp | ⚠ Δ -8.75pp from sequential, CI 包含 0 |
| Random cusip-null | n/a | ✓ p<0.001 |
| DSR | ✓ implied (V18 通过 V85) | ✓ p=0.033 |
| **CPCV with purge** | ✓ | **✗ PBO 0.43–0.57** |
| Drop-COVID + SST | ✓ | ⚠ 边缘 |
| **Verdict** | **deployable** | **borderline, 不建议 live capital** |

---

## 5. 我们走过的方法论坑

### 5.1 Sequential filter selection 的隐式 grid

V125→V136→V140 这条路径上，每加一个 filter 都"基于前一步看起来 OK" 决定是否保留。**每一步的隐式自由度乘起来 = 一个比单步看到的大得多的 implicit grid**。V146 unified grid 就是把这个 implicit grid 显式化的工具。

### 5.2 CSCV 不等于 CPCV

CSCV (Bailey 2014) 是多重测试惩罚 (selection bias 检查)。它解决的是"我比较了 N 个策略，最好的那个会有多少 selection inflation"。

CPCV (López de Prado 2018) 多了 purge gap，解决的是"label window 跨 train/test 边界的泄露"。

**这两个是独立的失败模式**。一个策略可以通过 CSCV 但是 fail CPCV (V146-best 就是这个例子)，反之亦然。

### 5.3 单一 robust metric 不够

我们过去会报"Full / Drop-COVID / SST" 三档作为 robustness。但这只是 sample-period robustness，不解决 selection bias 也不解决 leakage。**需要 sample-period × selection × leakage 三维 robustness。**

### 5.4 Visual selection 的诱惑

V143 时候我看着 fixed-weight 表选了 "balanced" 90/10 mix — 这其实是 Bug #2 第二次。**任何"在表上挑一个看起来好的"动作都是 OOS-peek**。要么用正式选择标准 (per-window nested CV)，要么完全 fixed (像 V150 那样 hardcode 50/50 然后看 industry-standard 测试)。

---

## 6. Strategy Graveyard：哪些死了，怎么死的

| 策略 | 看起来的 alpha | 死后真实 alpha | 死因 |
|---|---|---|---|
| V21B4 small-fund mid-cap | +12pp | ~+0pp | Bug #4 split adjustment |
| V26 mega-ensemble | Sharpe 2.27 | Sharpe 1.48 | Bug #2 OOS-peek |
| V62 small-cap specialist | +13.54pp | +4.83pp (loses to V18) | Bug #7 nested CV |
| V70 first-appearance | +9pp 单窗 | regression to mean | Bug #5 rolling outlier |
| V80 blind ensemble | optimization gain | actual loss | Bug #6 correlated duds |
| V125 STRICT | +8pp 单窗 | -8pp shrinkage | Bug #7 |
| V140 (STRICT + F144 + 10b5-1) | **+13.58pp** | **+5pp, CI 含 0** | Bug #7 sequential bias |
| V146-best | +14.5pp fixed-weight | borderline | Label leakage (CPCV fail) |
| V149 short Form 144 | mixed | weak | 信号弱 |

**Pattern**: 几乎每个 small-cap 信号都呈现"看起来漂亮 → 严格审计后衰减 8-10pp"的模式。这个模式本身就是 evidence — small-cap insider-buy data 里的真实 alpha 远小于 naive backtest 显示的水平。

---

## 7. V18 是怎么活下来的？

V18 不靠运气活下来的。它有结构性 features 让它对每个 bug 都不敏感：

1. **Mega-cap heavy** → 几乎没有 delisting (Bug #1 不咬)
2. **Top-7 fund 是 in-sample InfoRatio rank** → 选 fund 这一步在 IS only (Bug #2 自然防御)
3. **No SPY fallback in V13/V17b code path** → 空仓直接 cash (Bug #3 不存在)
4. **Mega-cap 不需要 split adjustment 重做** → Bug #4 不咬
5. **Long sample (16 yrs), 多 regime** → Bug #5 不容易
6. **V13 + V17b-1 correlation 0.65 (moderate)** → 不是 correlated duds (Bug #6)
7. **Hyperparameter grid 不大 (top-N=7, top-K=3, consensus=2 是简单参数)** → Bug #7 shrinkage 仅 -0.37pp
8. **Hold = 季度 (90 天)，与 quarterly block 完美 align** → 没有 label leakage (Bug #8)

**V18 的简洁性是它鲁棒性的来源。** 反过来，V146-best 在 7 维 grid (972 configs) + 长 hold (180d) + alt-data filter 的复杂性，是它 fragile 的根源。

---

## 8. 当前状态 & 下一步选项

### 8.1 现状 (2026-04-27)

✅ **V18 deployable**: +8.78pp/yr unbiased，通过全部 8 层审计
❌ **Phase 2 small-cap overlay**: 全部候选都失败
🤔 **V146-best 这个 borderline case**: 取决于你信不信 CPCV 的 purge 严格性

### 8.2 下一步可能方向

**A. 把 V18 deploy 起来** (推荐)
- 接进 ResearchDesk portfolio
- 写 paper trading runner
- 把 Bailey-López de Prado 三件套 (DSR + CSCV + CPCV) 抽成可复用模块，给以后所有策略用

**B. 重审 V146-best：CPCV 是不是过严** ← 用户想做的
- 测 block-size 敏感性 (S=4/8/16)
- 测 walk-forward 而非 CPCV (放弃组合数 fold，看真实 sequential train→OOS)
- 测 nominal lift 在不同 purge 下的稳定性而非 PBO 阈值
- 看 V146-best 在 short-hold (90d) 加更强 cluster (≥5 buyers + ≥$1M) 能不能保住 signal

**C. Pivot 到完全不同的 signal family**
- Options flow (gamma exposure, dealer positioning)
- 13F + alt-data (satellite, credit card, app downloads)
- Factor-style (quality, low-vol, momentum) 而不是 stock-picking
- Macro-driven sleeve (term structure, credit spread regime)

**D. 把 13F follower 的 V18 当 baseline，改造它去捕捉 small-cap 暴露 而不是用 V146-best 加层**
- Top-7 fund 里加一个"专门有 small-cap mandate 的 fund"
- 13F 数据本身的 small-cap 13F持仓 (而不是 Form 4 cluster)
- 这是回到 13F-only 数据，但做一个不同的 sleeve

### 8.3 我个人建议

短期 (1–2 weeks):
1. **B**：花 ~3 天重审 CPCV，搞清 V146-best 是不是真死了。如果发现 CPCV 在我们的样本规模下确实过严 → V146-best 可以小权重上 (10–15%)。
2. **A 并行**：开始把 V18 接进 ResearchDesk paper trading（不依赖 B 的结果）。

中期 (1–2 months):
3. 如果 B 救活了 V146-best → 部署 V18 + 15% V146-best
4. 如果 B 没救活 → **C**：开始一个新的 signal family，不要在 small-cap insider-buy 上继续打转

---

## 9. 重要文件索引

### Scripts (按里程碑)
- `scripts/v13_engine.py` — V13 mega-cap consensus
- `scripts/v17b_high_conviction.py` — V17b-1 高信念过滤
- `scripts/v18_*.py` — V18 50/50 mix
- `scripts/v85_v18_unified_grid.py` — V18 Bug #7 audit (-0.37pp)
- `scripts/v97b_net_insider_fast.py` — Form 4 cluster builder
- `scripts/v104_survivorship_check.py` — quarterly returns with imputation
- `scripts/v125_strict_audit.py` — STRICT spec, Bug #7 first crisis
- `scripts/v140_*.py` — Form 144 + 10b5-1 path (sequentially biased)
- `scripts/v146_unified_grid.py` — **The decisive Bug #7 test**
- `scripts/v147_v146_best_null.py` — V146-best random null
- `scripts/v150_v18_v146_fixed_weight.py` — fixed-weight backtest
- `scripts/v151_dsr_pbo.py` — Deflated Sharpe Ratio
- `scripts/v153_pbo_distinct.py` — PBO via CSCV
- `scripts/v154_cpcv_purged.py` — **CPCV with purge (V146-best 翻车)**
- `scripts/v155_short_hold_cpcv.py` — short-hold rescue attempt

### Reports
- `bugs.md` — bug log (Bug #1–#7, 需要更新加 #8)
- `V13_FINAL_REPORT.md` — V13 baseline 总结
- `V146_DETAIL.md` — V146 详细说明 (这次新增)
- `V155_FINAL_VERDICT.md` — V155 honest verdict
- `PROJECT_LOGBOOK.md` — 本文档
- GitHub Pages: https://businessmeetsprogramming.github.io/pages/13f-follower/

### Cache (data)
- `data/cache/v44_v3_quarterly.parquet` — V18 quarterly returns 基准
- `data/cache/v97_form4_buy_sell_wrds.parquet` — Form 4 全量
- `data/cache/v131_form144_wrds.parquet` — Form 144 raw
- `data/cache/v140_rule10b5_wrds.parquet` — 10b5-1 mapping
- `data/cache/v146_unified_grid.parquet` — V146 nested CV results

---

## 10. Appendix: 用户给过的关键反馈

记录用户在过程中给的几次 turning-point 反馈：

1. "我觉得不够全面，你应该再次做一个 full grid search，然后 synthesizing everything together" → 触发 V146
2. "你要研究一下，我们的 Bug 7 是一个很重要的问题。那么过去大家在 develop strategy 的时候，是怎么去避免 Bug 7 的？" → 触发引入 DSR / CSCV / CPCV
3. "我们可能老卡在 Bug 7 上，所以我们要看看大家有什么新的逻辑来避免这个事情" → 强化方法论提级
4. "行，我们继续尝试这个东西" → V154 / V155
5. "我觉得还好，我们先把 V18 做好，再看怎么做小盘股" → 当前节点：deploy V18 优先
