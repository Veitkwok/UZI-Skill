---
description: 完整深度分析一只股票（22 维数据 + 51 位大佬量化评委 + 17 种机构分析方法 + 杀猪盘检测 + Bloomberg 风格 HTML 报告）
argument-hint: "[股票名称或代码，例如 华工科技 / 002273 / AAPL / 00700.HK]"
---

# 深度分析任务

用户输入: $ARGUMENTS

## 执行流程（两段式 · 你必须在中间介入）

### 第一段 · 数据采集 + 骨架分（脚本完成）

```bash
cd <plugin_root>
pip install -r requirements.txt 2>/dev/null
cd skills/deep-analysis/scripts
python -c "from run_real_test import stage1; stage1('$ARGUMENTS')"
```

这会跑完 Task 1 → 1.5 → 2 → 3（规则引擎骨架分），输出到 `.cache/{ticker}/` 下。

### 第二段 · 你来分析（核心！不能跳过！）

Stage 1 跑完后，**你必须做以下事情**：

**1. 读取评委骨架分**

读 `.cache/{ticker}/panel.json`，看 51 人各自打了多少分。特别关注：
- Top 5 看多和 Top 5 看空分别是谁？他们的 headline 有没有说服力？
- 有多少人 skip 了？（非 A 股时游资会 skip）
- 有没有明显不合理的分数？

**2. 逐组分析（spawn 4 个并行 sub-agent）**

对每组投资者，spawn 一个 Agent：

**Agent 1 · 价值 + 成长派（10 人）**
```
你要扮演巴菲特/格雷厄姆/费雪/芒格/邓普顿/卡拉曼/林奇/欧奈尔/蒂尔/木头姐，
逐一对 {stock_name} ({ticker}) 给出判断。

公司数据：{从 raw_data.json 摘取关键数据}
规则引擎参考分：{从 panel.json 摘取这 10 人的 score/headline}
真实持仓：{巴菲特持有苹果/BYD, 段永平持有苹果/茅台/腾讯 等}

对每人输出: investor_id, signal, score(0-100), headline(引用数字), reasoning(2-3句)
你可以覆盖规则引擎的分数——你是在模拟这个人的判断，不是跑公式。
```

**Agent 2 · 宏观 + 技术派（9 人）**
**Agent 3 · 中国价投 + 量化（9 人）**
**Agent 4 · 游资（23 人）** — 非 A 股直接全部 skip

**3. 合并 agent 结果**

把 4 个 agent 返回的 {signal, score, headline, reasoning} 覆盖到 `.cache/{ticker}/panel.json` 的对应投资者上。

**4. 写你的定性评语**

对关键维度（财报/估值/护城河/行业）写 1-2 句定性评语。如果需要，web search 补充信息。

### 第三段 · 生成报告（脚本完成）

```bash
python -c "from run_real_test import stage2; stage2('$ARGUMENTS')"
```

这会读取你更新后的 panel.json + 数据，生成最终 HTML 报告。

### 第四段 · 向用户汇报

1. 综合评分 + 定调
2. 51 评委投票分布
3. DCF 内在价值 vs 当前价
4. Top 3 看多理由 + Top 3 看空理由
5. Great Divide 金句
6. 杀猪盘等级
7. 报告文件路径

## 快速模式（跳过 agent 介入）

如果用户说"快速分析"或"不用那么详细"：
```bash
cd <plugin_root>
python run.py $ARGUMENTS --no-browser
```
这会 stage1 + stage2 一把跑完，不做 agent 分析。

## 禁止

- 不跑脚本就编造数据
- 跳过 agent 分析直接出报告（除非用户明确要快速模式）
- 用"基本面良好"等模板话术
