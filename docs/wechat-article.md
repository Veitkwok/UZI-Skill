# 巴菲特和赵老哥坐在同一张桌子上：51 位投资大佬塞进 Claude，连"自查报告数据对不对"都机械强制，这个国人开源 Skill 火了

36 小时 610 星，106 次 fork。

有人干了件 Buffett-skills 那个项目想都不敢想的事：**不是一个投资人，是 51 个**——从巴菲特到段永平，从索罗斯到赵老哥，从 Simons 到拉萨天团，用每个人自己的方法论同时给你的票打分。然后更狠的是：跑完之后 agent 必须自己核对一遍所有内容，有问题物理上发不出报告。

![](https://img.shields.io/github/stars/wbh604/UZI-Skill?style=social)

不是语录，不是模板话术，不是"巴菲特会怎么看这只票"那种 LLM 瞎编。是每个人**真实的决策规则**——巴菲特 180 条量化规则、赵老哥 T+2 到 T+5 打板纪律、Simons "信号 Sharpe 跌破 0.5 就撤"的模型心法——全部写进代码。

说实话我看到作者把巴菲特和赵老哥列在同一张桌子上那一刻，就觉得这个项目有搞头。

## 01 它是什么

**UZI-Skill**（中文名 "游资"）是一个开源的 Claude Code Plugin，全市场（A 股 / 港股 / 美股）深度分析引擎。

GitHub：https://github.com/wbh604/UZI-Skill

> 一句话：输入一只股票，Claude 变成你的私人分析师，跑完 22 维数据 + 51 个大佬量化评委 + 17 种华尔街机构级分析方法，最后吐一份 600KB 的 Bloomberg 风格报告 + 朋友圈竖图 + 群聊战报。

### 1.1 为什么不只做一个人

Buffett-skills 做了一件漂亮的事：把巴菲特的思维框架炼成 Skill。但投资市场里巴菲特不是唯一的答案。

- 你分析阿里巴巴——Munger 2021 年也分析过阿里，然后亏了九位数。**"连他都看走眼的票，你凭什么只用他一个人的框架？"**
- 你分析小市值打板龙头——巴菲特根本不碰，他的规则会直接给你 skip。那该听谁的？
- 你分析量化因子股——价值派规则全错位，得听 Simons 的模型思维。

所以作者做了一件更狠的事：**51 个投资者，每个人用自己的方法论同时给同一只票打分。** 意见打架就打架，分歧本身就是信号。

### 1.2 七大流派分工

| 组 | 风格 | 人数 | 代表人物 |
|---|---|---|---|
| A | 经典价值 | 6 | 巴菲特 · 格雷厄姆 · 芒格 · 费雪 · 邓普顿 · 卡拉曼 |
| B | 成长投资 | 4 | 林奇 · 欧奈尔 · 蒂尔 · 木头姐 |
| C | 宏观对冲 | 5 | 索罗斯 · 达里奥 · 霍华德马克斯 · 德鲁肯米勒 · 罗伯逊 |
| D | 技术趋势 | 4 | 利弗莫尔 · 米内尔维尼 · 达瓦斯 · 江恩 |
| E | 中国价投 | 6 | 段永平 · 张坤 · 朱少醒 · 谢治宇 · 冯柳 · 邓晓峰 |
| F | A 股游资 | 23 | 章盟主 · 赵老哥 · 炒股养家 · 佛山无影脚 · 拉萨天团 … |
| G | 量化系统 | 3 | 西蒙斯 · 索普 · 大卫·肖 |

## 02 核心架构

### 2.1 两段式 pipeline

这个项目最像工程的地方——不是把所有活儿丢给 LLM，而是**脚本和 agent 分工**：

```
Stage 1 (脚本)          → 数据采集 · 22 维 · 20+ 免费源并发
        ⏸️ Agent 介入   → 读数据 → role-play 51 评委 → 写判断
Stage 2 (脚本)          → 综合研判 · 风格加权 · 报告生成
        🛡 机械级自查    → 13 条检查 · critical 不过物理 block
Task 5 (脚本)          → Bloomberg 风格 HTML + 分享卡 + 战报
```

脚本负责事实层，agent 负责定性层，**中间用 HARD-GATE 强制 agent 不能跳过**。

### 2.2 22 维数据从哪来

不是 GPT wrapper，是真的把数据源拉穿了：

| 数据 | 主源 | 兜底 |
|---|---|---|
| 实时行情 / PE / 市值 | 东方财富 push2 | 雪球 → 腾讯 → 新浪 → 百度 |
| 财报历史 | akshare | 雪球 f10 |
| K 线（A/H/U 三市场） | akshare 东财 | Sina → yfinance |
| 龙虎榜 / 北向 / 两融 | akshare | 东财 data 子域 |
| 研报 / 公告 | 巨潮 cninfo + akshare | 同花顺 |
| 港股财报 | `stock_financial_hk_analysis_indicator_em` | — |
| 宏观 / 政策 / 舆情 | DuckDuckGo 权威域 site: 查询 | — |

全部免费源，**零 API key**。妙想 API 可选（免费申请）。

### 2.3 因地制宜：每个评委回答自己的 3 个问题

这是和 Buffett-skills 最大的区别：

| 投资者 | 时间框架 | 翻盘条件 |
|---|---|---|
| **巴菲特** | 10 年以上 / 永远 | ROE 连续 2 年跌破 12% · CEO 离职且战略转向 |
| **赵老哥** | T+2 到 T+5 | 板上砸盘 · 龙头断板 · 量能跟不上 |
| **Simons** | 平均持仓 < 2 天 | 信号 Sharpe 跌破 0.5 · 因子衰减 |
| **Lynch** | 公司故事讲完为止，3-5 年 | PEG > 2 · 库存/应收增速超营收 |
| **Soros** | 反身性循环一轮 · 随时可翻转 | 市场停止验证我的叙事 |
| **冯柳** | 3-6 个月等错杀修复 | 基本面证伪（订单/产能/客户反馈） |

不是给所有人加 3 个同样的模板字段，是每人按**自己的方法论**填。

### 2.4 原话库，不瞎编

Buffett-skills 的语料来自巴菲特股东信语料库 49 个概念页面——挺好，但只有他一个人。

UZI 做了更重的活：派了 4 个并行 research agent 去取证，给 45+ 投资者建了 639 行的 `quotes-knowledge-base.md`——每人 3-5 条原话，**全部带可点 URL 溯源**。

```
### 巴菲特 (buffett) · Berkshire Hathaway Letters
核心方法论: 以合理价格买入优秀企业并长期持有

1. "别人贪婪时我恐惧..." — 2004 年致股东信 (berkshirehathaway.com/letters/2004ltr.pdf)
2. "用合理的价格买一家好公司..." — 1989 年致股东信
3. "我们最喜欢的持有期是永远。" — 1988 年致股东信
4. "只有在退潮时，你才知道谁在裸泳。" — 2001 年致股东信
```

Agent 生成评语前**必读**这个文件。每一句话都能点开看原文，不瞎编。

## 03 v2.9 的杀手锏：机械级自查 gate

这是我看完整个项目最拍案的部分。

### 3.1 软要求不够

过往版本 SKILL.md 里有 `HARD-GATE-FINAL-CHECK`——"软要求"，写在文档里让 agent 自觉遵守。

**问题是：agent 可能跳过、可能忘、可能做半截。**

有个真实 bug 是这样被用户发现的：用户分析云铝股份（000807.SZ），属于工业金属铝行业，报告里却归类为"农副食品加工"。跑完全流程报告都发出去了才有人发现错了。

根因：老代码用 `str.contains(industry[:2])` 对证监会行业分类做 fuzzy 匹配，"工业金属"→"工业"→命中 4 个含"工业"子串的行业，`iloc[0]` 盲选了第一个"农副食品加工业"。

这就是软 gate 的代价。**agent 没做最后核查，用户替他做了。**

### 3.2 v2.9 改成机械强制

`lib/self_review.py` 13 条自动检查，每条都对应一个历史 BUG 的教训：

| severity | check | 抓什么 |
|---|---|---|
| 🔴 critical | check_industry_mapping_sanity | 行业碰撞（工业金属→农副食品加工这种） |
| 🔴 critical | check_all_dims_exist | wave2 超时丢失维度 |
| 🔴 critical | check_empty_dims | fetcher crash / timeout 的空维度 |
| 🔴 critical | check_hk_kline_populated | 港股 kline 数据源挂掉 |
| 🔴 critical | check_hk_financials_populated | 港股财报是空 stub |
| 🔴 critical | check_panel_non_empty | 51 评委全 skip / 分数异常 |
| 🔴 critical | check_coverage_threshold | 数据完整性 < 60% |
| 🔴 critical | check_placeholder_strings | synthesis 含 "[脚本占位]" |
| 🔴 critical | check_agent_analysis_exists | agent_analysis.json 缺失或未 review |
| 🟡 warning | check_valuation_sanity | DCF / Comps 全 0 |
| 🟡 warning | check_metals_materials_populated | 金属股 materials 空 |
| 🟡 warning | check_industry_data_coverage | 7_industry 定性字段需 agent 补 |
| 🟡 warning | check_factcheck_redflags | 编造"苹果产业链"但 raw_data 无证据 |

然后关键的机械化：

```python
# assemble_report.py · v2.9
review = review_all(ticker)
if review["critical_count"] > 0:
    raise RuntimeError("⛔ BLOCKED by self-review")
# 过了才能继续拼 HTML
```

**物理上无法发出一份坏掉的报告。** 直到 agent 修完所有 critical issue。

### 3.3 迭代流程

```
loop:
  1. python review_stage_output.py <ticker>
  2. 读 .cache/<ticker>/_review_issues.json
  3. if critical > 0:
       对每条 issue 执行 suggested_fix（补数据 / 重跑 / 写 agent_analysis 覆盖）
       重跑 review
  4. critical == 0 才进入 HTML 生成
```

每次新 BUG 修完，对应的 `check_*` 规则都会加到 self_review。**下次同类 bug 跑完就自动抓到，不再靠用户反馈。**

## 04 实测亮点

### 4.1 一个报告长什么样

分析水晶光电（002273.SZ）的真实产出：

**综合评分**：61.8 / 100 · "关注"
- 费雪 100 分看多："这家在产业链口碑不错"
- 卡拉曼 0 分看空："无 30% 安全边际"
- 木头姐 100 分看多："AR/VR 平台拐点"
- 格雷厄姆 32 分看空："PE × PB 超 22.5"

**51 评委聊天室**：每个人用自己语言风格发言，引用命中规则。

**DCF 估值 5×5 敏感性热力图**：WACC 6.96% · 内在价值 ¥20.73 · 安全边际 -28.6%

**IC 投委会备忘录 三情景回报**：Bull ¥26.95 · Base ¥20.73 · Bear ¥14.51，各带概率和假设。

**22 维深度卡**：K 线蜡烛 + PE Band + 雷达图 + 供应链流程图 + 温度计 + 环形图。

**朋友圈竖图**：1080×1920，一键分享。

### 4.2 17 种机构方法不是吹的

从 [anthropics/financial-services-plugins](https://github.com/anthropics/financial-services-plugins) 移植并适配 A 股参数：

**估值建模**：DCF · Comps · 3 表预测 · Quick LBO · 并购增厚/摊薄
**研究工作流**：首次覆盖 · 财报 beat/miss · 催化剂日历 · 投资逻辑追踪 · 晨报 · 量化筛选 · 行业综述
**深度决策**：IC 投委会备忘录 · Porter 五力 + BCG · DD 尽调清单 · 单位经济学 · 价值创造计划 · 组合再平衡

Anthropic 原版是 US-only 且要 FactSet 付费数据。**这里全免费源跑 A 股。**

### 4.3 杀猪盘检测

这个是 A 股特色功能。8 个信号扫描：

- 大量低质量账号同时推荐
- 推荐话术模板化
- 付费社群 / VIP 直播间引流
- 基本面与热度脱节（ST 股突然暴涨）
- K 线异常配合（直线拉升）
- 老师 / 股神人设推广
- 跨平台联动（小红书 + 抖音 + 微信群）
- 虚假研报 / 伪造消息

朋友"群里老师带"的票输进去，红绿灯 🟢🟡🟠🔴 直接告诉你该跑路还是能看。

## 05 从巴菲特到芒格都躲不过中国股票

Charlie Munger 2021 年通过 DJCO 重仓阿里巴巴（BABA），2022 年 ~70% 回撤后不得不砍仓一半。[2022 年 DJCO 年会上他称"这是我犯过最糟糕的错误之一"](https://www.cnbc.com/2023/02/15/charlie-munger-says-he-regrets-alibaba-investment-one-of-the-worst-mistakes.html)，估计九位数损失。

**连一位史上最伟大投资者之一的人，都在中国市场的监管和竞争格局上走了眼。**

作者在英文版 README 里写了这么一句："Names this plugin helps you understand: Alibaba (BABA/09988.HK), Tencent (00700.HK), Kweichow Moutai (600519.SH), CATL (300750.SZ), BYD (002594.SZ), Pop Mart (09992.HK), Pinduoduo (PDD) — **the same names that keep showing up in Western portfolios and keep surprising their owners** 😉"

免责声明尾部还有一句我特别喜欢：
> "Charlie Munger still lost money on Alibaba, and he actually read the 10-Q. Invest at your own risk."

## 06 怎么上手

不管你用什么 agent，**丢一句话过去就行**：

### Claude Code

```
/plugin marketplace add wbh604/UZI-Skill
/plugin install stock-deep-analyzer@uzi-skill
```

装好后说 `/analyze-stock 贵州茅台` 或 `/analyze-stock 00700.HK`。

### Codex / OpenClaw / Cursor / Gemini / Windsurf / Devin

每家 agent 都有对应安装命令。懒得看的话就丢这句：
> 克隆 https://github.com/wbh604/UZI-Skill，读 AGENTS.md，帮我深度分析 贵州茅台。

### 📱 不在电脑前？

对任何 agent 说：
> 分析 00700.HK，用远程模式，生成一个公网链接让我手机能看。

它会自动 `--remote` 启动 Cloudflare Tunnel，给你一个 `https://xxx.trycloudflare.com`。

### CLI

```bash
git clone https://github.com/wbh604/UZI-Skill
cd UZI-Skill && pip install -r requirements.txt
python skills/deep-analysis/scripts/run_real_test.py 600519
```

## 写在最后

Buffett-skills 做的事是漂亮的——把一个人的思维框架炼成 Skill。

UZI 做的事更重也更诚实：**承认没有一个人能独自搞定市场**。巴菲特看走眼阿里巴巴，索罗斯看走眼反身性会反转，赵老哥看走眼大盘系统性风险。但 51 个人同时看一只票，分歧本身就是最重要的信号。

**然后 v2.9 加的那个机械级自查 gate，更是把"AI 工具不靠谱"这个行业通病按在地上。**

过往 LLM 工具的典型路径是：用户反馈 bug → 作者修 → 用户再反馈下一个 → 再修 → 周而复始。UZI 的路径是：每次修完 bug，对应的检查规则永久进自查引擎，下次同类 bug 跑完就自动抓到。**靠机器替 agent 记住 bug 记忆。**

投资里少犯错就赢了大多数人。工具里，能让每次运行都自动核对一遍自己产出的内容，就赢了市面上 95% 的 GPT wrapper。

GitHub：https://github.com/wbh604/UZI-Skill
