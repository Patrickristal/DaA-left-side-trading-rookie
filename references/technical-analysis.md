# M2 — 个股技术面分析 (Single-Stock Technical Analysis)

这个模块给 Patrick 一个"严格但易学"的技术面分析层。Patrick 自评 TA 词汇有限，所以本模块的设计哲学是**用一组固定命名的因子代替散文式描述** — 反复使用同一组名字，让 Patrick 通过重复建立词汇库，而不是每次都见到新术语。

读完 SKILL.md 后再读这份。本模块借鉴 [Zeeklog 综述](https://zeeklog.com) 中 Chinese stock analysis 项目普遍使用的**命名因子注册表 (named factor registry)** 模式 + [vnpy](https://github.com/vnpy/vnpy) 的 ArrayManager 滚动窗口抽象 + Mark Minervini 的 Stage 2 / SEPA + 缠论三类买卖点。

---

## 模块契约 (Module Contract)

- **Standalone trigger**：Patrick 给一个或多个 ticker / 股票名称 + "看一下"、"分析一下"、"K 线怎么样"、"现在这个位置"、"能不能进"、"该不该减"；或上传 K 线截图。
- **Composition role**：作为增强模块嵌入 M1 / M3 / M4 时，提供 `技术面卡片` — 4-6 行精简版（趋势阶段 + 关键因子读数 + 一句话观察）。
- **Inputs needed**：
  - 必需：标的代码或名称
  - 强烈推荐：时间维度（日线 / 周线 / 月线，默认日线）、关注点（趋势 / 关键位 / 形态 / 强度对比）
  - 可选：Patrick 当前持仓状态（持有 / 观望 / 已平），用于调整深度
- **Outputs produced**（标准小节，其他模块可引用）：
  - `## 标的快照` — 当前价、当日涨跌、所属板块、近 5 日表现
  - `## 趋势阶段` — Weinstein 4 阶段判定 + 中文映射
  - `## 命名因子读数` — 25 个固定因子的当前值 + 状态标记
  - `## 量价配合 & 形态` — 量价关系 + 关键 K 线形态识别
  - `## 关键位` — 支撑 / 阻力 / 突破点 / 关键均线
  - `## 板块相对强度` — 标的 vs 所属板块 vs 沪深300 / 恒指
  - `## 缠论视角`（可选）— 当前位于第几买点（仅当 Patrick 显式要求 / 已有缠论上下文）
  - `## 综合判断 & 风险点` — 综合 stage + 关键观察因子 + 风险点
- **Common compositions**：
  - **+ M4（论点验证）**：M4 调用 M2 提供 `技术面证据` 一句话 + 数据
  - **+ M6（资金面叠加）**：在 `综合判断` 前嵌入 M6 三桶资金读数
  - **+ M5（政策面叠加）**：在 `综合判断` 前嵌入 M5 政策催化背景
  - **嵌入 M1**：作为 `你的雷达` 中某只票的展开切片

---

## 触发场景

Patrick 通常这样开场：
- "**腾讯**现在这个位置看一下" / "**中芯国际** K 线怎么样"
- "**阳光电源**还能进吗" / "**理想**这个回调到位了吗"
- "帮我看下 **600519**" / "**00700** 周线"
- 上传一张 K 线截图 + "这个图你怎么看"
- "**储能板块龙头**给我做技术面排序"（多标的）

---

## 设计原则

### 1. 命名因子注册表 — 用同一组词汇反复出现
Patrick 看到的不是 "MACD 显示动能减弱"，而是 **`MACD_hist_zero_cross_age = 3 ⚠`** — 一个固定名字 + 一个值 + 一个状态符号。
- 同一只票每次分析都使用同一组因子名
- 反复出现 → Patrick 自然记住每个名字含义
- 状态符号统一：`✅` 健康 / `⚠` 警告 / `❌` 破损 / `↗` 改善 / `↘` 恶化 / `→` 持平
- 不要发明新术语 — 注册表里没有的因子不要使用

### 2. Stage 优先于 Signal
Weinstein 4 阶段（Stage 1 底部盘整 / Stage 2 上升 / Stage 3 顶部盘整 / Stage 4 下降）是顶层判断。任何信号都先放在 stage 框架里：
- Stage 2 + 量能放大 → 突破有效
- Stage 4 + 量能放大 → 加速下跌
- Stage 1 末期 + 量能放大 → 启动信号
- 不告诉 Patrick "MACD 金叉" 而不告诉他这只票在 Stage 几 — 单纯信号没有意义

### 3. 引用可信来源，不当 expert
Patrick 是 TA 新手，他不需要技能装专家。需要的时候明确引用：
- "Mark Minervini 的 SEPA 把 RS Rating ≥ 80 作为 Stage 2 必要条件"
- "缠论的第三类买卖点定义为 [...]，源自缠中说禅原文 [...]"
- "William O'Neil 的 CANSLIM 'L' (Leader) 要求板块内 RS Top 5%"
- 不知道的就说 "这条规则我不能确认是哪本书原文，建议你查证"

### 4. 不喊点位，不下指令
- ❌ "目标价 50 元"
- ❌ "建议在 45 元买入"
- ❌ "应该立刻平仓"
- ✅ "当前价 47.2 元，距 60 日均线（45.8 元）+3.1%，距 20 日新高（49.5 元）-4.6%"
- ✅ "若跌破 60 日均线（45.8 元），趋势阶段从 Stage 2 退化为 Stage 3 的概率显著上升"

### 5. 周线 / 日线 / 30 分钟看不同问题
默认日线。但要主动判断哪个周期更回答 Patrick 的问题：
- 中线持仓判断 → 周线 + 日线
- 短期买点 → 日线 + 30 分钟
- 趋势阶段判定 → **必须**周线（Stage 在周线最稳定）
- 多周期不一致时（日线 Stage 2 但周线 Stage 3） — 明确指出，不平滑掉

---

## 命名因子注册表

Patrick 应该把这张表当词汇表反复看。25 个因子分 5 组。

### A. 趋势阶段 (Stage)

| 因子名 | 计算 | 值域 | 健康 | 警告 |
|---|---|---|---|---|
| `weinstein_stage` | 周线 30MA 方向 + 价格位置 | 1/2/3/4 | 2 | 3 / 4 |
| `MA20_slope_5d` | MA20 最近 5 日斜率（年化%） | 数值 | > +30% | < 0% |
| `MA60_slope_20d` | MA60 最近 20 日斜率 | 数值 | > +10% | < 0% |
| `MA_alignment_daily` | 5 / 10 / 20 / 60 日多头排列状态 | full/partial/broken/inverse | full（5>10>20>60 全部上行） | broken / inverse |
| `MA_alignment_weekly` | 周线 5 / 10 / 20 / 60 排列 | 同上 | 同上 | 同上 |

### B. 价格位置 (Position)

| 因子名 | 计算 | 值域 | 健康 | 警告 |
|---|---|---|---|---|
| `price_vs_MA20_pct` | (价 - MA20) / MA20 | % | +0% ~ +10% | < -3% 或 > +20% |
| `price_vs_MA60_pct` | (价 - MA60) / MA60 | % | +5% ~ +30% | < 0% |
| `price_to_52w_high_pct` | 距 52 周高百分比 | % | -5% 到 -25% | < -40% |
| `price_to_52w_low_pct` | 距 52 周低百分比 | % | > +30% | < +10% |
| `consec_up_days` | 连续上涨天数 | int | 1 - 4 | ≥ 7 (过热) |

### C. 量价 (Volume-Price)

| 因子名 | 计算 | 值域 | 健康 | 警告 |
|---|---|---|---|---|
| `vol_ratio_5_20` | 5 日均量 / 20 日均量 | ratio | 1.0 - 1.5（温和放量） | > 2.5（异动） / < 0.6（缩量） |
| `vol_price_consistency_5d` | 5 日量价同向率 | -1 ~ +1 | > +0.5 | < -0.3（背离） |
| `turnover_pct_60d` | 当日换手率 60 日分位 | 0-100 | 30 - 80 | < 10 (无人问津) / > 90（过热） |
| `volume_breakout_signal` | 突破时量比 ≥ 2.0 + 价格新高 | bool + value | True，且突破有效 | False（突破未配合放量） |

### D. 动能 (Momentum)

| 因子名 | 计算 | 值域 | 健康 | 警告 |
|---|---|---|---|---|
| `RSI14_daily` | 14 日 RSI | 0-100 | 40 - 70 | > 80（超买） / < 30（超卖） |
| `MACD_hist_daily` | MACD 柱状值 | 数值 | > 0 上升 | < 0 下降 |
| `MACD_hist_sign_change_age` | MACD 柱状值最近反向天数 | int | < 10（趋势刚启动） | 反向 1-2 天内（注意确认） |
| `MACD_divergence` | MACD 与价格背离 | none/top/bottom | none | top（顶背离）/ bottom（底背离） |
| `KDJ_state` | KDJ 当前位置 | low/mid/high/golden/death | golden cross 低位 | death cross 高位 |

### E. 波动与强度 (Volatility & Strength)

| 因子名 | 计算 | 值域 | 健康 | 警告 |
|---|---|---|---|---|
| `BOLL_position_20d` | 价格在布林带的位置 | 0 - 1 | 0.4 - 0.7 | < 0.1 / > 0.9 |
| `BOLL_band_width_pctl_60d` | 布林带宽度 60 日分位 | 0-100 | 30 - 70 | < 15（极度收敛） / > 85（极度扩张） |
| `ATR14_pct` | ATR / 价格 | % | 1.5 - 4 | > 6（剧烈波动） |
| `RS_vs_sector_5d` | 5 日相对板块强度（5 日 / 板块 5 日 - 1） | % | > +0% | < -3% |
| `RS_vs_sector_20d` | 20 日相对板块强度 | % | > +0% | < -5% |
| `sector_RS_rank_in_market` | 所属板块在全市场板块涨跌幅排名 | rank | top 30% | bottom 30% |

### F. 形态 (Patterns) — 仅识别基础 4 个

不展开缠论 / 复杂 K 线学。只识别：

| 因子名 | 计算 | 值 |
|---|---|---|
| `pattern_basic` | 当前 K 线形态 | hammer / engulfing_bullish / engulfing_bearish / doji_at_extreme / none |
| `pivot_breakout_state` | Minervini pivot point 突破状态 | building_base / breakout / breakout_failed / extended / none |
| `chan_buy_point`（缠论，可选）| 当前缠论买卖点 | 1B / 2B / 3B / 1S / 2S / 3S / none |

---

## 输出模板（standalone）

```
# 个股技术面：[股票名称] [代码]

## 1. 标的快照
- 当前价：47.20（+1.4% / 2026-05-01 收盘）
- 所属板块：申万二级 - 储能设备 / 概念板块 - 锂电池、新型储能
- 近 5 日：+8.2%（板块同期 +5.6%，跑赢板块 +2.6pp）
- 主要财务：PE_TTM 32x（行业中位数 28x），PB 4.1x（5 年 65% 分位）

## 2. 趋势阶段 (Weinstein)

**当前 stage**：**Stage 2** ✅（上升趋势确立）

依据：
- `weinstein_stage = 2`
- `MA60_slope_20d = +18%` ✅
- `MA_alignment_weekly = full`（周线 5>10>20>60 全部上行）✅
- 价格高于周线 30MA 8.4%

注释：Stage 2 持续 [N] 周，距上次 Stage 1 → 2 转换 [日期]。

## 3. 命名因子读数（核心 12 个）

| 维度 | 因子 | 读数 | 状态 |
|---|---|---|---|
| 趋势 | weinstein_stage | 2 | ✅ |
| 趋势 | MA20_slope_5d | +35% (年化) | ✅ |
| 位置 | price_vs_MA20_pct | +5.1% | ✅ |
| 位置 | price_to_52w_high_pct | -3.2% | ✅ |
| 量价 | vol_ratio_5_20 | 1.6 | ✅ 温和放量 |
| 量价 | vol_price_consistency_5d | +0.72 | ✅ 量价同向 |
| 量价 | turnover_pct_60d | 78 | ✅（接近上沿） |
| 动能 | RSI14_daily | 68 | ⚠ 接近超买 |
| 动能 | MACD_hist_daily | +0.32, 上升中 | ✅ |
| 强度 | RS_vs_sector_20d | +6.8% | ✅ |
| 强度 | sector_RS_rank_in_market | 板块全市场 #4 | ✅ |
| 形态 | pivot_breakout_state | breakout | ✅ 5 日内突破 |

（其余因子读数省略，如需完整 25 个请说"展开全部因子"）

## 4. 量价配合 & 形态

- **量价**：5 日量比 1.6 + 价格同向（`vol_price_consistency_5d = +0.72`）= 健康放量上行
- **关键 K 线**：5 日前出现 `engulfing_bullish`（多头吞没形态），位于 60MA 上方 + 量比 1.8，确认突破信号
- **pivot point**：Minervini pivot 形成于 [日期] 价位 [N]，5 日内有效突破

## 5. 关键位

| 位类型 | 价格 | 含义 |
|---|---|---|
| 上方阻力 1 | 49.50 | 52 周新高 |
| 上方阻力 2 | 53.20 | 2024 年高点 |
| 当前价 | 47.20 | — |
| 下方支撑 1 | 45.80 | 60 日均线 |
| 下方支撑 2 | 43.50 | 突破前的横盘平台上沿 |
| Stage 退化警戒 | 42.00 | 跌破即 Stage 2 → 3 |

## 6. 板块相对强度

- 标的 5 日 +8.2% vs 板块 +5.6% vs 沪深300 +1.2%
- `RS_vs_sector_5d = +2.6pp`，`RS_vs_sector_20d = +6.8pp`
- 所属板块在全市场板块 5 日涨跌幅 **排名 #4**（共 31 个二级板块）
- 在板块内涨幅排序：**第 2**（板块龙头嫌疑，详见 M6 联合分析）

> 综合判断：板块强势 + 个股板块内排名靠前 = 龙头嫌疑成立。Patrick 之前提到的"板块对了选股次了" 问题在此票上**不成立**。

## 7. 缠论视角（可选）

（仅当 Patrick 显式要求或有缠论上下文时输出）

- 当前缠论结构：日线段于 [日期] 创新高，未结束
- 当前买卖点：`chan_buy_point = 3B`（第三类买点已在 [日期] 出现，价位 [N]）
- 引用：缠论第三类买卖点定义见缠中说禅原文（缠中说禅 49 课）

## 8. 综合判断 & 风险点

**Stage 2 + 健康放量 + 板块龙头嫌疑 + Pivot 突破有效**。技术面整体偏多。

**关键观察因子**（以下任一翻转就要重新评估）：
1. `MA20_slope_5d` 转负 → 短期动能熄火
2. 跌破 45.80（60 日均线） → Stage 2 → 3 转换
3. `vol_price_consistency_5d` 跌至 0 以下 → 量价开始背离
4. `RS_vs_sector_20d` 跌至负值 → 龙头地位丢失

**风险点**：
- `RSI14_daily = 68` 接近 70 超买阈值，短期回调概率上升
- `BOLL_position_20d = 0.78` 在布林带上沿附近，向上空间被压缩
- `turnover_pct_60d = 78` 偏高，存在 Patrick 自己提到的"过早过热"风险

---

如要登记此股为活跃 trend integrity 监测，建议条件 = 上述"关键观察因子" 4 条。如确认请说"登记此票"，将由 M4 接手写入 memory。
```

---

## 简化版本（作为增强模块嵌入时使用）

当 M2 嵌入 M1 / M3 / M4 时，输出仅 4-6 行：

```
**M2 技术面卡片（[股票名] [代码]）**
- Stage：[N]（依据 [一句]）
- 关键因子：[3 个最相关的，含读数和状态符号]
- 量价：[一句话]
- 强度：板块内排名 #N，RS_vs_sector_20d = [%]
- 一句话观察：[20 字以内]
- ⚠ 关键观察因子：[最容易翻转的 1-2 条]
```

---

## 数据获取

```python
# scripts/ta_factors.py 提供入口
from scripts.ta_factors import compute_factors

result = compute_factors(
    symbol="600519",  # 或 HK 代码 "00700"
    market="A" or "HK",
    period="daily",  # daily / weekly / 30min
    factors="all" or ["weinstein_stage", "MA20_slope_5d", ...]
)
# 返回 dict: {factor_name: {value, status, threshold_health, threshold_warn}}
```

底层调用：
- `ak.stock_zh_a_hist(symbol, period, start_date, end_date, adjust="qfq")` 获取 A 股 OHLCV
- `ak.stock_hk_hist(symbol, period, start_date, end_date, adjust="qfq")` 获取港股 OHLCV
- `pandas-ta` 或 `talib` 计算 MA / MACD / RSI / KDJ / BOLL / ATR
- `ak.stock_board_industry_name_em()` + `ak.stock_board_industry_hist_em()` 获取板块基准
- `ak.stock_individual_info_em()` 获取标的所属板块

数据始终用 `adjust="qfq"`（前复权）做技术面分析。

---

## 反模式

- ❌ **散文式描述代替命名因子** — "MACD 显示动能减弱" 不算分析。必须 `MACD_hist_daily = -0.18, MACD_hist_sign_change_age = 2 ⚠`
- ❌ **不报 Stage 直接谈信号** — "金叉了" 在 Stage 4 是死叉反弹陷阱，在 Stage 1 是启动信号，意义完全相反。必须先报 stage
- ❌ **多周期不一致时藏着** — 日线 Stage 2 但周线 Stage 3 一定要明说
- ❌ **形态识别不基于事实** — 凭印象说 "看起来像头肩顶"。形态识别必须基于因子注册表里的 `pattern_basic` 输出，识别不出来就说 none
- ❌ **缠论强行附加** — Patrick 没要求时不主动展开。缠论是可选层不是必选层
- ❌ **板块对比省略** — 个股技术面不能脱离板块判断。任何分析必须有 `RS_vs_sector_*` 一行
- ❌ **不附风险点** — 即使 stage 2 + 全因子健康，也要给"什么会让我改变判断" 1-2 条
- ❌ **目标价 / 建议买入** — 见原则 4

---

## 输出长度参考

- Standalone 完整分析：默认 700-1000 字
- 嵌入式简化卡片：60-120 字
- 多标的对比（如龙头排序）：每只 60-100 字 + 一张排序表

---

## 何时应该问 Patrick 而不是自己决定

- Patrick 给一个名字但有歧义（"小米" — 小米集团 01810 还是小米汽车专题？）：直接问
- 周期不明确（"看一下"）：默认日线但提一句 "默认日线，要看周线请说"
- 标的不在数据源里 / 是新股不足 60 个交易日：明说 "数据不足，因子计算需要 ≥ 60 个交易日"
- 截图分辨率太低看不清：直接问 "这张图分辨率不够，能传一张更清晰的或告诉我代码我自己拉数据"

---

## 与可信来源的引用规范

需要引用具体方法论时，使用以下格式：

> **Mark Minervini 的 SEPA Stage 2 标准**（出处：*Trade Like a Stock Market Wizard*，2013）：要求 RS Rating ≥ 80，价格高于 150MA 高于 200MA，200MA 上行 ≥ 1 个月。

> **William O'Neil 的 CANSLIM-L (Leader)**（出处：*How to Make Money in Stocks*）：板块内 RS 排名前 5%。

> **缠中说禅第三类买卖点**（出处：缠中说禅博客原文 49 课）：定义为 [...]

不能确认出处的方法论，明说"这是市场常见说法但我不能确认原始出处"，不要硬编。
