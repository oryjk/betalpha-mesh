---
name: betalpha-mesh-skills
description: Betalpha Mesh 投研技能集入口：邀请码注册与 MCP 客户端配置（betalpha-mesh-onboarding）、资产买卖信号解读（betalpha-asset-signal）、因子血缘查询与解释（betalpha-factor-lineage），以及投研场景技能（主动基金星级评估、ETF综合评估、红利风格轮动）。当用户想接入 Betalpha Mesh、询问某资产的买卖信号/多空方向、查询某因子的构成与方法论、或触发已发布投研场景（基金评级、ETF评级、红利风格信号）时，路由到对应子技能执行。服务地址 https://mesh.betalpha.com。
---

# Betalpha Mesh 技能集（路由入口）

本仓库是一个技能集合 plugin，本文件是集合入口：先判断用户问题属于哪个子技能，
再用 Skill 工具加载对应技能并按其流程执行，不要在本文件里直接展开子技能的流程。

| 用户问题 | 子技能 |
| --- | --- |
| 有 `bmi_` 邀请码，想注册账号 / 配置 MCP 连接 | [betalpha-mesh-onboarding](skills/betalpha-mesh-onboarding/SKILL.md) |
| 某基金/股票/ETF 的买卖信号、多空方向、"该买还是该卖" | [betalpha-asset-signal](skills/betalpha-asset-signal/SKILL.md) |
| 某因子怎么计算、由什么构成、方法论是什么、为什么是这个值 | [betalpha-factor-lineage](skills/betalpha-factor-lineage/SKILL.md) |
| 某只主动权益基金的星级评估、综合评估结果 | [betalpha-active-fund-star-rating](skills/betalpha-active-fund-star-rating/SKILL.md) |
| 某只权益型ETF的综合评估、星级查询 | [betalpha-etf-comprehensive-score](skills/betalpha-etf-comprehensive-score/SKILL.md) |
| 核心红利/类债红利/周期红利的买卖信号、红利组合持仓 | [betalpha-dividend-style-rotation](skills/betalpha-dividend-style-rotation/SKILL.md) |

都不匹配时：引导用户先完成接入（onboarding），或用免 Key 的
`search_factors` / `search_assets` / `search_macro_indicators` 自由探索
Betalpha Mesh 有哪些因子、资产与宏观指标；也可用 `search_scenario_skills`
搜索其他已发布场景并经 `explain_scenario_skill` 获取结构化执行计划。

## 通用规则（对所有子技能生效）

- **T-1 数据**：一切"最新"指最近一个已同步数据的交易日（`as_of_date`），
  不是提问当天；
- **服务地址**：默认 `https://mesh.betalpha.com`（MCP 与 REST 同进程同端口，
  MCP 端点 `/mcp`）；
- **配额**：快照/时序/血缘/解释类工具需要 `X-API-Key` 并消耗每日配额，
  搜索与元数据类工具免 Key 免配额。
