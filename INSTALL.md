# Installation Guide

跨多个 Claude agent 的安装路径。挑你用的那个。

---

## 1. Claude Cowork (macOS desktop app)

最简单 — Cowork 原生支持 `.skill` 格式。

### 步骤

1. 从本仓库的 [Releases](../../releases) 页面下载最新 `da-a-hk-trading-strategist.skill`，或自己打包：
   ```bash
   git clone https://github.com/Patrickristal/DaA-left-side-trading-rookie.git
   cd da-a-hk-trading-strategist
   # 用仓库内提供的 Python 脚本打包，或手动 zip
   ```
2. 打开 Cowork → Settings → Skills → "Install from file"
3. 选择刚下载的 `.skill` 文件
4. 重启 Cowork 或新建对话，技能会自动加载

### 卸载

Cowork → Settings → Skills → 点 da-a-hk-trading-strategist → Uninstall

---

## 2. Claude Code (CLI)

### 步骤

```bash
# 直接 clone 到 Claude Code 的 skills 目录
git clone https://github.com/Patrickristal/DaA-left-side-trading-rookie.git \
    ~/.claude/skills/da-a-hk-trading-strategist

# 安装 Python 依赖（让脚本能调真实数据）
pip install -r ~/.claude/skills/da-a-hk-trading-strategist/requirements.txt
```

验证：

```bash
ls ~/.claude/skills/da-a-hk-trading-strategist/SKILL.md
# 应该看到 SKILL.md 文件
```

### 更新

```bash
cd ~/.claude/skills/da-a-hk-trading-strategist
git pull
```

---

## 3. Claude Agent SDK（自己写的 agent）

SDK 没有内置 skill 加载机制，需要手动注入。两种方式：

### 方式 A：注入 SKILL.md 到 system prompt

```python
from anthropic import Anthropic

client = Anthropic()
with open("/path/to/da-a-hk-trading-strategist/SKILL.md") as f:
    skill_md = f.read()

system_prompt = f"""You are Patrick's trading strategist.

When the user discusses A-share / HK markets, follow this skill:

{skill_md}
"""

response = client.messages.create(
    model="claude-sonnet-4-6",
    system=system_prompt,
    messages=[{"role": "user", "content": "看一下储能板块"}],
)
```

### 方式 B：让 agent 按需读 reference 子文件

给 agent 一个 file-reading 工具，agent 会按 SKILL.md 的 routing 指示读 `references/<mode>.md`：

```python
# Pseudo - depends on your agent framework
agent.add_tool("read_file", read_file_handler)
agent.set_skill_path("/path/to/da-a-hk-trading-strategist")
agent.run("盘前看一下半导体板块")
# Agent reads SKILL.md → identifies M1+M5 modules → reads
# references/premarket-recap.md + references/policy-analysis.md
# → produces structured output
```

---

## 4. Python 依赖

让脚本能真正调数据需要：

```bash
pip install -r requirements.txt
```

### 不装能不能用？

**能** — 技能在没有 AKShare 的环境下会优雅降级为 **框架式分析**：
- M2 不再算 22 个因子，但仍按命名因子词汇结构化输出
- M6 不再拉北向 / 主力数据，但仍呈现 3 桶 × 3 维度框架
- M4 / M5 完全独立于数据，照常工作

适合纯文本分析、教学、demo 场景。

---

## 5. 跨 agent 共享 memory

如果你在多个 agent（Cowork + Claude Code + 自己的 SDK）都用本技能，想让 forecast 留痕和论点登记**统一**：

```bash
# 在每个 agent 启动前 export 一次
export DA_A_HK_MEMORY_DIR="$HOME/Documents/da-a-hk-memory"
export DA_A_HK_CACHE_DIR="$HOME/Documents/da-a-hk-cache"
```

`scripts/memory_store.py` 和 `scripts/akshare_helpers.py` 都 honor 这两个环境变量。

`memory/` 目录默认会被 `.gitignore` 排除（它是个人交易数据，不应进版本控制）。

---

## 6. 第一次校准

装完后第一次使用，建议这样开始：

### 第一次截图复盘（M3 校准 — 仅东方证券用户）

```
你：[上传一张你常用的东方证券持仓页截图] 帮我看看这个截图字段对不对
```

技能会按 `assets/dongfang_screenshot_layout.md` 的字段位置反问你确认。校准结果会写入该文件。

### 第一次论点登记（M4）

```
你：我觉得 [某板块/某股] 接下来 [时间窗] 会 [方向]，主要逻辑是 [...]
```

技能会跑 3 阶段对抗辩论 + 立场矩阵，最后让你确认是否登记 4-5 条 trend-break triggers 写入 `memory/active_theses.jsonl`。

### 后续日常

```
你：盘前看一下          → M1 + 自动检查所有 active 论点的 trigger
你：复盘下今天          → M1（盘后版）+ trigger 状态变化
你：[上传成交截图] 复盘 → M3 + 与论点对照 + 经验标签累积
你：盯紧 [方向] 政策    → 创建周期跟踪任务
```

---

## 7. 卸载 / 清理

### 卸载技能

```bash
# Cowork: 通过 Settings → Skills 卸载
# Claude Code:
rm -rf ~/.claude/skills/da-a-hk-trading-strategist
```

### 清理 memory（如要全部重来）

```bash
# 慎用 — 会删掉所有论点登记 + 复盘记录 + forecast 留痕
rm -rf ~/.claude/skills/da-a-hk-trading-strategist/memory/
# 或者你设的 DA_A_HK_MEMORY_DIR 路径
```

### 清理 cache

```bash
rm -rf ~/.cache/da-a-hk-trading
```

---

## 故障排查

### "AKShare not installed" 错误

```bash
pip install akshare>=1.18
```

### 港股数据不准 / 缺失

港股精度本来就低于 A 股，AKShare 是 15 分钟延迟，主力数据精度有限。建议盘中实时数据用券商 app 截图给技能。

### Xueqiu fetch 失败

雪球需要 cookies。设环境变量后重试：

```bash
export XUEQIU_COOKIE="xq_a_token=...; xqat=...; xq_r_token=..."
```

cookies 从浏览器开发者工具复制。

### 技能没自动触发

- Cowork：检查 Settings → Skills 列表里 da-a-hk-trading-strategist 是否启用
- Claude Code：确认 `~/.claude/skills/` 目录下 SKILL.md frontmatter 完整
- 任何 agent：开新对话（已有对话的 system prompt 是会话开始时锁定的，新装的技能不会注入到旧会话）

---

## 报告 issue

[GitHub Issues](../../issues) 欢迎反馈：
- 触发不灵 → 提供你输入的 prompt + 期望模块组合
- 数据获取失败 → 提供 stack trace + AKShare 版本
- 输出格式问题 → 提供具体例子 + 期望改进
