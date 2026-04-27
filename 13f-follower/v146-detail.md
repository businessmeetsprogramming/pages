# V146 详解 — 统一网格搜索 (Unified Grid Search)

**Date**: 2026-04-27
**File**: `scripts/v146_unified_grid.py` (212 lines)
**Output**: `data/cache/v146_unified_grid.parquet`

---

## 1. V146 是什么？一句话回答

**V146 是把所有 Phase 2 small-cap insider-buy 信号的全部维度，一次性放进同一个 grid 里，让 nested CV 在 972 个配置中"自由选"——这是判断 V125/V136/V140 这条 sequential 路径是不是 Bug #7 偏差的"决定性测试"。**

如果 V140 (Form 144 + 10b5-1) 是真的，统一 grid 应该一致选出 V140-like 的配置，performance 接近原报告。如果是 sequential 选择偏差，统一 grid 会选出五花八门的配置，Δ 会大幅缩水。

**结果：Δ 从 V140 sequential 报告的 +13.58pp 缩到 V146 unified 的 ~+5pp，95% CI [-3.00, +12.40]，包含 0。** → V140 sequential 是部分偏差。

---

## 2. 为什么要做 V146？— 我们怎么走到这一步的

### 2.1 V125 STRICT 出问题

V125 测了一个简单的 spec：micro<$300M + ≥3 unique insiders + $50K net buy + 30d window + 180d hold。看起来是个干净的"小盘 insider cluster"信号。但它在 nested CV 下塌了 ~8pp。

### 2.2 加 filter 试图救活它（这就是 Bug #7 的温床）

为了"修复" STRICT，我们 sequential 地加了几个过滤层：

```
V125 STRICT
  ↓ + 加 Form 144 reverse filter (insiders 没有近期卖出意图)
V136
  ↓ + 排除 10b5-1 自动卖出
V140
```

V140 报告的 Full Δ +13.58pp，10/0 win/loss，CI [+8.63, +18.32]。**看起来是个 home-run，CI 远远不含 0。**

### 2.3 Sequential 选择的隐患

问题是：每加一个 filter，我们都在"看着 OOS 效果"决定要不要保留它。这等价于在 OOS 上做了多步选择。即使每一步看起来都是 in-sample 决定，**整条路径上累计的 selection 自由度远超我们以为的 1 个 hyperparameter**。

形式化地说：sequential layering of filters 等价于在一个**比单步 grid 大得多的隐式 grid** 上做了 max-of-grid 选择，但只把"被选中"那一条路径的 nested CV 报上来。这是 Bug #7 的变种。

### 2.4 V146 就是为了破解这个隐式 grid

把 sequential path 上每一步的"选 or 不选"都变成 grid 维度，让 nested CV 一次性看到全部组合。如果 V140-like 配置真的是结构性最优，那它在统一 grid 里也会被一致选中。

---

## 3. V146 grid 的 7 个维度 (972 个 config)

```
mcap_band       × min_buyer × min_value × f144_filter × exclude_10b5_1 × quality       × hold_period
3 (all/<$300M/  × 3 (2/3/5) × 3 ($10K/  × 3 (None/    × 2 (no/yes)     × 2 (None/      × 3 (60/90/180d)
   $100-300M)               $50K/$250K)   60d/180d)                       positive_ROA)

3 × 3 × 3 × 3 × 2 × 2 × 3 = 972 configs
```

每一维的含义：

| 维度 | 选项 | 意思 |
|---|---|---|
| `mcap_band` | all / micro<$300M / $100M-$300M | 股票市值 universe |
| `min_buyer` | 2 / 3 / 5 | 30天 cluster 内最少几个不同 insider P-buy |
| `min_value` | $10K / $50K / $250K | 30天 cluster net-buy 美元下限 |
| `f144_filter` | None / 60d / 180d | "Form 144 reverse" — 过去 N 天没人 file Form 144 (无近期卖出意图) |
| `exclude_10b5_1` | False / True | 是否过滤掉 10b5-1 算法卖出 (V140 自带 filter) |
| `quality` | None / positive_ROA | 是否要求 Compustat ROA > 0 |
| `hold_period` | 60 / 90 / 180 days | 持有期 |

加上每窗 5 档 V18 mix 权重 `[0.0, 0.1, 0.2, 0.3, 0.5]`，每个 held-out window 实际比较 `972 × 5 = 4,860` 种 candidate strategy。

---

## 4. V146 的实验流程 (per-window nested CV)

```
对每个 held-out window (2014-15, 2015-16, ..., 2023-24, 共 10 个):

  IS years   = [2010, ..., held_start - 4]  (训练用)
  TUNE years = [held_start-3, held_start-2, held_start-1]  (调参用)
  HELD years = [held_start, held_start+1]  (评估用，这一段只看一次)

  for cfg in 972 config_panels:
      for w in [0.0, 0.1, 0.2, 0.3, 0.5]:
          mix = (1-w) * V18 + w * cfg.strat
          score = sharpe(mix on IS years) + sharpe(mix on TUNE years)
          # 注意：score 完全不看 HELD years

  best_cfg, best_w = argmax(score)
  # 仅在 HELD years 上算 alpha
  nested_alpha = cagr(mix_with_best on HELD) - cagr(SPY on HELD)
  v18_alpha    = cagr(V18 on HELD)            - cagr(SPY on HELD)
  Δ = nested_alpha - v18_alpha   # 这就是这个 window 的 unbiased lift
```

10 个 windows 跑完 → 10 个 Δ → bootstrap CI → 报 Full / DropCOVID / SST 三种 robustness。

**关键点：选择 cfg+w 完全发生在 IS+TUNE 上。HELD 只用来读 alpha。这是"公平的 hyperparameter sweep"。**

---

## 5. V146 的实验结果

### 5.1 Aggregate

| 路径 | Full Δ | 95% CI | W/L | DropCOVID Δ | SST Δ |
|---|---|---|---|---|---|
| **V140 sequential (旧报告)** | +13.58pp | [+8.63, +18.32] | 10/0 | +13.31pp | +12.13pp |
| **V146 unified (公平测)** | **+5pp** (大约) | **[-3.00, +12.40]** | 较低 | 衰减 | 衰减 |

**Δ 缩水 ~8.75pp。CI 现在包含 0。** 这就是 sequential bias 的尺寸。

### 5.2 选中 config 的 diversity

V146 的 nested CV 在 10 个 windows 里**没有**一致选 V140-like 配置。具体哪个 config 被选最多次：

> V146 unified grid 在 9/10 个 windows 里选了 `positive_ROA` quality filter。
> 最常被选的整套 spec 是：`micro<$300M, ≥3 buyers, $250K, no f144 filter, no 10b5-1 exclude, positive_ROA, 180d hold`。

这个就是后来被称为 **"V146-best"** 的 locked spec。**注意：它是 post-hoc 选出来的，不是 V140**。Form 144 filter 在 unified grid 下 ≥9/10 个窗口被弃用，10b5-1 排除也是。

### 5.3 V146-best 这个 spec (locked)

| 维度 | V146-best 选择 |
|---|---|
| Universe | micro < $300M |
| Min buyers | ≥3 unique insiders |
| Min net buy | ≥ $250,000 |
| Window | 30 days (固定) |
| Form 144 filter | ❌ **不用** (V140 用了，在统一 grid 下被 reject) |
| 10b5-1 exclude | ❌ **不用** (同上) |
| Quality | positive ROA (Compustat) |
| Hold | 180 days |

也就是说：**V140 加的两个"alt-data filter"（Form 144 reverse + 10b5-1 exclude）在 unified grid 下并没有提升 nested CV score，是 sequential 路径上的 selection artifact**。真正在数据里的信号是 micro-cap + ≥3 insiders + 中等到大的 net buy + ROA>0 + 长 hold。

---

## 6. V146 之后我们做了什么？— V147~V155

### 6.1 V147 — Lock V146-best spec, 跑 random null

把 V146-best 当成固定 spec（没有任何 grid 选择），用 cusip-shuffle 做 200 次 null。

结果：Drop-COVID p<0.001, SST p=0.005。**signal 在 right tail**，统计上不为 0。

### 6.2 V150 — Fixed-weight V18 + V146-best

每窗都用相同权重 mix (不再 per-window 选权重)：

| Strategy | CAGR | Sharpe | MaxDD | α SPY |
|---|---|---|---|---|
| V18 alone | +20.3% | 1.01 | -36% | +5.5pp |
| V18 + 30% V146-best | +26.0% | 1.52 | -21% | +11.2pp |
| **V18 + 50% V146-best** | **+29.3%** | **1.69** | **-17%** | **+14.5pp** |

这个看起来非常漂亮。

### 6.3 V151 — Deflated Sharpe Ratio (DSR)

Bailey-López de Prado 多重测试惩罚。N=5832 个 trials → DSR=0.967, p=0.033。**通过 5%。**

### 6.4 V152/V153 — PBO via CSCV

Bailey 2014. T=48 quarters 切 10 段，C(10,5)=252 train/test combos。

V153 (10 个 distinct strategies)：**PBO = 0.163，远低于 0.5 的阈值。** V18+50% V146-best 在 135/252 (54%) 的 combo 里赢得 IS-best。

**到这一步，看起来 V146-best 通过了所有 industry-standard tests。**

### 6.5 V154 — CPCV with purge gap (这里翻车)

López de Prado 2018. CPCV 比 CSCV 多了一个 **purge band** — 在 train/test 边界插入"留空区"，防止 forward-return window 跨越边界泄露 label。

我们的 hold = 180 天 ≈ 2 quarters，所以 quarter q 的 strategy return 实际上用到了 q+2 的价格。adjacent quarters 共享信息。CSCV 没意识到这个，CPCV 必须 purge ≥2 quarters。

| Purge | PBO | 判断 |
|---|---|---|
| 0 quarter (= V153 CSCV) | 0.143 | PASS |
| 1 quarter | 0.429 | borderline |
| **2 quarters (leakage-matched)** | **0.571** | **FAIL** |

也就是说：V153 的 0.163 是因为没 purge label overlap 才好看。一旦正确 purge，PBO 上升到 0.43–0.57。

### 6.6 V155 — 短 hold 救援尝试 (失败)

如果 hold 缩到 60–90 天，purge=1 就够覆盖 leakage 了。但是：

| Hold | Solo Sharpe | V18+50% Sharpe | CPCV PBO | V18+50% IS-best wins |
|---|---|---|---|---|
| 60d  | 0.557 | 1.221 | 0.393 (PASS) | 8/28 (29%) |
| 90d  | 0.648 | 1.158 | 0.393 (PASS) | 3/28 (11%) |
| 180d | 1.381 | 1.689 | 0.43–0.57 | varies |

**短 hold 下 leakage-clean，但 solo Sharpe (0.55–0.65) 跟 cusip-shuffle 噪声 (0.57–0.63) 没法区分。** 90d 里 SPY 在 15/28 个 fold 里赢 IS-best — V146-best 连 SPY 都打不过。

**结论**：长 hold 下信号"看起来好"是因为 label leakage；短 hold 下没 leakage 信号就消失了。

---

## 7. V146 的两面性 — honest 解读

### 7.1 V146 做对了什么

1. **暴露了 V140 的 sequential bias**：从 +13.58pp 缩到 +5pp。这是真信号。
2. **找到了一个 spec (V146-best) 通过了 random null + DSR**：在某种意义上，确实有 some signal in there。
3. **方法论上是干净的**：每窗都从 972 个 config 里公平选一个。

### 7.2 V146 没做到的

1. **Nested CV 用的还是 quarterly block，没有 explicit purge**。这就是后来 CPCV 翻车的原因。
2. **V146 unified Δ +5pp 的 CI [-3, +12] 已经包含 0**——即使按 V146 自己的标准，信号是 borderline 的。我们之后报的"V146-best 通过 PBO 0.163"是过度乐观地解读。
3. **没有把"label leakage" 单独当作一个维度来检查**。直到 V154 才意识到 hold ≥ block_size 这件事必须 purge。

---

## 8. CPCV 是不是太严格了？— 你的问题

这是一个开放问题，值得好好想想。下面把 pro/con 列清楚：

### 8.1 支持"CPCV 太严格"的论点

- **Purge=2 quarters 在 T=48 quarters 上很 aggressive**：每个 fold 把 4 个 quarter (test 2 + purge 2) 拿掉，只剩 ~4 个 quarter 训练，样本数太少导致 train Sharpe 不稳，IS-best 容易随机漂移。
- **Block-size 选择本身有自由度**：S=8 / S=10 / S=16 会给不同 PBO。我们没系统地测 block-size 敏感性。
- **CPCV 的"50% baseline"假设所有 fold 同等重要**：实际中我们更关心晚近年份（2020-2024 vs 2010-2014），均匀加权 fold 可能低估晚近 OOS 表现。
- **Purge 机制是为高频策略设计的**：18 年里只有 ~64 个 quarterly observation，CPCV 对小样本季频策略可能本就不可靠。
- **V146-best 通过了 random null (p<0.001 drop-COVID) + DSR (p=0.033)**：这两个是**独立**的失败模式 (selection bias / label leakage)。CPCV 反对的是 leakage，不否定 random null 显示的"信号确实存在"。

### 8.2 支持"CPCV 是对的"的论点

- **180 天 hold 跨 2 quarters 在数学上 100% 真实**：q 季度的 strategy return 真的用了 q+2 的价格，这不是估计问题，是结构事实。
- **CSCV/PBO=0.163 → CPCV/PBO=0.57 的差距太大**：这个 swing 不是噪声，是真实的 leakage 量级。
- **V146 自己的 unified Δ +5pp CI 已经过 0**：CPCV 只是更精确地刻画了同一个 borderline 现象。
- **V18 alone 不靠 CPCV 就过**：这说明 CPCV 不是普遍 over-strict 工具，是只对 hold-长 strategies 严格。

### 8.3 可能的折衷

如果觉得 CPCV purge=2 过严，可以考虑：

1. **报 multiple purge levels**：让 reader 看 0/1/2 三档。我们已经这么做了。
2. **降 hold 到 90d 但提高 cluster 信号强度**：V155 显示 90d 信号弱。但如果可以**通过更强的 cluster 标准（比如 ≥5 buyers + ≥$1M）让短 hold 信号回升**，这是值得测的。
3. **Walk-forward 而非 CPCV**：放弃组合数所有 fold，只看真实 sequential train→OOS。这样 leakage 仍然要处理（drop adjacent quarter），但不会被 C(8,2)=28 个 fold 的方差冲垮。

---

## 9. 我对 V146 的当前看法 (honest)

**V146 是这个项目最重要的诚实时刻**。它把 V125→V136→V140 这条 sequential 路径上 +8.75pp 的 selection bias 完整暴露出来。即使 V146-best 后来在 V154/V155 又挂了，**V146 本身仍然是好的方法学**。

**V146 的一个真实贡献**：发现了 Form 144 filter + 10b5-1 exclude 在 unified grid 下都被 reject 了——这是 V140 那条路径的"真相"。

**V146-best 这个 spec**：是个 borderline candidate。在 light testing (DSR, random null) 下过，在 heavy testing (CPCV with purge) 下不过。是不是真信号取决于你信哪一边的方法论。

**从 deployment 角度**：
- 如果你信 CPCV：V146-best 不应该上 live capital
- 如果你怀疑 CPCV 过严：可以小权重 (10–15%) 试一试，但要明白这是个 calibrated bet on methodology choice
