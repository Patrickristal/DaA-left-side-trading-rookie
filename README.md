# da-a-hk-trading-strategist

> Claude 技能：为 A 股 / 港股的中线持仓交易者做策略研究、技术分析、资金流向解读、政策影响判断、盘前盘后复盘、交易日记复盘和投资观点压力测试 — 不替你下单，做你的策略师 + 多元视角 second opinion。

> A Claude skill that turns Claude into a strategist + mentor for retail traders operating in China A-share (大 A) and Hong Kong markets. Multi-module composable analysis (pre/post-market briefing, technical analysis, trade journal review, thesis verification, policy analysis, money-flow tracking) backed by Tetlock's superforecasting methodology. Default Chinese output. Strategist role only — never executes trades.

***

## 这是什么

这是一份**Claude Skill**（不是普通 Python 库，也不是交易机器人）。装到任何 Claude agent（Cowork / Claude Code / Claude Agent SDK）后，当对话涉及 A 股 / 港股 / 板块 / 资金 / 政策 / 个股 / 截图复盘时，Claude 会自动按本技能的方法论响应。

### 设计哲学

- **策略师不是执行者**：本技能不下买卖单、不喊点位 — 提供分析、可观测事实、多元视角，决策权在用户
- **可追溯的预测**：不拒绝预测（拒绝预测的策略师没有价值），但每个预测必须满足三条门槛 — 源可追溯 / 情景化（base/bull/bear）/ 可证伪（带翻转条件）
- **机械化事实优于自由裁量**：可量化的（RS 排名、北向净流入、均线位置）用具体数字和阈值；判断性的明确标注"判断"
- **Trend integrity 监测，不强制纪律**：登记论点的趋势条件，每天勾选状态；条件破损时报告事实，决策权交还
- **多源交叉，不单一信号**：板块龙头识别用 6 维交叉（板块内 RS + 主力 + 北向 + 龙虎榜知名游资 + 机构席位 + 跑赢板块均值）

***

## 六大能力模块

模块独立可用，也可按提示词自由组合：

| ID     | 模块                   | 主要触发                    | 核心创新                                                 |
| ------ | -------------------- | ----------------------- | ---------------------------------------------------- |
| **M1** | 盘前 / 盘后 prep & recap | "盘前看一下"、"复盘"、时段词        | Trend integrity 网格 + 龙头识别表（多源交叉）                     |
| **M2** | 个股技术面分析              | 给一只票 + "看一下/分析" + K 线截图 | **22 个命名因子注册表**（解决 TA 词汇问题）                          |
| **M3** | 交易日记 & 持仓复盘          | 上传券商截图 + "复盘"           | **12 个经验标签 taxonomy** + 与 M4 论点登记闭环                  |
| **M4** | 观点验证 & 思路深挖          | "我觉得"、"反驳一下我"、"还有什么角度"  | **3 阶段对抗辩论** + **5×3 立场矩阵** + trend-break trigger 登记 |
| **M5** | 政策影响分析               | 提到具体政策 / 文件 / 会议        | **5 类政策分类** + 时点提取 + 受益板块 lookup                     |
| **M6** | 三资金趋势分析（量 / 向 / 速）   | "看资金"、"主力"、"北向"         | **3 桶 × 3 维度网格** + FLOW\_REGIME\_CHANGE 跨桶共振警报       |

### 组合矩阵示例

| 用户提示             | 主模块 | 增强模块         |
| ---------------- | --- | ------------ |
| "今晚给我准备明天看半导体板块" | M1  | M5 + M6 + M2 |
| "中芯国际现在能进吗"      | M2  | M6 + M5      |
| "我觉得储能要起来"       | M4  | M6 + M2      |
| "复盘下我昨天那笔" + 截图  | M3  | M2           |

***

## 预测方法论：Tetlock 10 角度框架

`references/forecasting-discipline.md` 是技能里**第二重要的文件**（仅次于 SKILL.md）— 任何带预测的输出都必须遵守。三层结构：

**第一层 — 信号源（基于什么）**

- \#1 政策驱动 + 兑现时间点（如 4 月 17 日发改委 1.755 万亿 6 月底前下达）
- \#3 资金面 / 流动性信号（北向 / 两融 / ETF / 龙虎榜 / 央行 OMO）
- \#4 基本面拐点 + 业绩预期差（光伏看硅料 / 白酒看动销 / AI 看 token 消耗）

**第二层 — 方法（用什么思维框架）**

- \#2 Reference Class Forecasting（参照类预测，超预测者最常用）
- \#5 多视角对抗（Actively Open-Minded Thinking，强制写正反双方）
- \#6 概率化 + 贝叶斯更新（每条新信息显式宣告概率调整方向 + 幅度 + 理由）
- \#7 Fermi 估计（拆解大问题为可估子问题）

**第三层 — 元规则（怎么让预测"诚实"）**

- \#8 边界条件与黑天鹅（前提 + 失效条件 + 黑天鹅风险）
- \#9 时间尺度区分（周/月/季/年/多年精度不同，超 2 年只给方向）
- \#10 事后复盘 / 校准（forecast 留痕 + Brier score + 月度按概率分桶诊断 over/under-confidence）

### 标准预测输出模板

每次正式预测包含：信号源 + Reference Class + Base/Bull/Bear 三档 + Fermi 拆解（≥季度）+ 边界条件 + 留痕 ID。

***

## 核心闭环

技能的灵魂是三个跨模块闭环（用 jsonl 文件物理实现）：

```
M4 论点登记 ─→ memory/active_theses.jsonl
                    │
                    ↓
M1 盘前/盘后 ─→ 每日检查 trigger 状态 ─→ 报告 trigger 破损
                    │
                    ↓
用户操作 ─→ 上传截图 ─→ M3 复盘
                    │
                    ↓
M3 复盘 ─→ 与 M4 登记对比 ─→ 写 memory/trade_log.jsonl
                    │
                    ↓
M3 检测"重复同样错误" ─→ 标签累计 ≥ 3 次提示结构性问题
                    │
                    ↓
M4 下次论点登记 ─→ 检索历史相似论点 + 经验标签 ─→ 提醒避坑
```

`memory/forecasts.jsonl` 同步留痕所有正式预测，月底自动跑 Brier score 校准报告。

***

## 安装

详见 [INSTALL.md](INSTALL.md)。简版：

**Cowork**：拖 `.skill` 文件到 Settings → Skills → Install from file

**Claude Code**：

```bash
git clone https://github.com/<你的用户名>/da-a-hk-trading-strategist.git ~/.claude/skills/da-a-hk-trading-strategist
```

**Claude Agent SDK**：解压后在 system prompt 注入 `SKILL.md` 内容。

**Python 依赖**（让脚本能真正调数据时需要）：

```bash
pip install -r requirements.txt
```

不装也能用 — 没数据时技能优雅降级为框架式分析。

***

## 项目结构

```
da-a-hk-trading-strategist/
├── SKILL.md                              ← 顶层路由 + 6 模块 + 组合矩阵 + 模块契约
├── references/                           ← 各模块详细方法论 (7 个文件)
│   ├── premarket-recap.md                M1
│   ├── technical-analysis.md             M2
│   ├── trade-journal-review.md           M3 (含东方证券截图字段映射)
│   ├── insight-verification.md           M4 (3 阶段辩论 + 立场矩阵)
│   ├── policy-analysis.md                M5
│   ├── money-flow-analysis.md            M6
│   └── forecasting-discipline.md         ★ Tetlock 10 角度框架
├── scripts/                              ← Python 实现 (8 个 + __init__)
│   ├── akshare_helpers.py                基础数据层 (符号归一化 + 缓存 + 错误信封)
│   ├── memory_store.py                   3 个 jsonl + Brier 校准报告
│   ├── ta_factors.py                     22 个命名因子计算
│   ├── fund_flow.py                      3 桶网格 + FLOW_REGIME_CHANGE 检测
│   ├── rs_ranker.py                      龙头识别 (6 项打分)
│   ├── policy_search.py                  5 类政策词典 + 时点+影响 lookup
│   ├── trend_integrity.py                M1 监测器 (7 类 trigger dispatch)
│   └── xueqiu_fetch.py                   雪球 degraded fallback
├── assets/
│   └── dongfang_screenshot_layout.md     东方证券字段映射 (4 页面类型)
└── examples/                             ← 测试样本和报告生成器
    ├── test-1-prediction.md              真实测试 prompt 的完整 markdown 输出
    ├── test-1-prediction-v2.pdf          ⭐ 16 页 PDF 排版示例（封面+目录+情景色编码）
    └── md_to_pdf.py                      Markdown → PDF 渲染器（含 CJK + 报告排版）
```

***

## 数据源

主要数据源是 [AKShare](https://github.com/akfamily/akshare)（Python 开源金融数据库）。技能 `scripts/akshare_helpers.py` 封装了 \~20 个常用接口，其他脚本通过它消费数据。

数据缺口（明说不编造）：

- 政治局会议 / 中央经济工作会议结构化日历 — AKShare 无，靠 `news_cctv` 近似
- 港股实时数据 — AKShare 是 15 分钟延迟
- L2 盘口 / 券商研报 — 没有
- Wind / Choice 一致预期 — 需用户上传

***

## 借鉴的方法论 & 工具

致谢以下框架和项目（详见 `references/forecasting-discipline.md` 引用文献）：

**预测方法论**

- Philip Tetlock & Dan Gardner. *Superforecasting: The Art and Science of Prediction*. Crown, 2015.
- 10 角度框架的三层结构（信号源 / 方法 / 元规则）由本技能用户 Patrick 提供。

**交易方法论**

- William J. O'Neil. *How to Make Money in Stocks*（CANSLIM-L 板块龙头标准）
- Mark Minervini. *Trade Like a Stock Market Wizard*（SEPA Stage 2 + Trend Template）
- Curtis Faith. *Way of the Turtle*（机械化 trailing stops + 海龟 80/20 研究）
- Howard Marks. *The Most Important Thing*（second-level thinking）
- Brett Steenbarger. *The Daily Trading Coach*

**开源项目**

- [AKShare](https://github.com/akfamily/akshare) — 主要数据源
- [TradingAgents-CN](https://github.com/hsliuping/TradingAgents-CN) — 多 agent 辩论架构灵感
- [vnpy](https://github.com/vnpy/vnpy) — 事件引擎 + 策略模板抽象
- [Microsoft qlib](https://github.com/microsoft/qlib) — 因子表达式 DSL 思想

***

## 免责声明

⚠ **本技能不是投资建议**

本项目提供的所有分析、预测、概率、板块判断仅为方法论演示和分析框架，**不构成任何投资建议、买卖推荐或财务咨询**。

- A 股 / 港股投资有重大风险，可能损失全部本金
- 过往业绩不代表未来表现
- 本技能基于公开数据 + 算法框架，不能替代专业投资顾问
- 用户应自行承担所有交易决策的全部责任

作者及贡献者**不对**因使用本技能产生的任何直接或间接损失负责。

***

## License

MIT License — 见 [LICENSE](LICENSE)。

***

## Contributing

PR 欢迎，特别是：

- 因子注册表扩展（`scripts/ta_factors.py` 加更多因子）
- 知名游资 / 机构席位字典完善（`scripts/rs_ranker.py`）
- 政策分类词典 + 行业影响 lookup 扩展（`scripts/policy_search.py`）
- 不同券商截图字段映射（`assets/`）
- 港股数据源补充（沽空比率 / 个股主力等 AKShare 缺口）

CHANGELOG 见 [CHANGELOG.md](CHANGELOG.md)。
