---
name: betalpha-factor-lineage
description: 查询与解释因子血缘：一个因子怎么计算、由哪些上游因子/组合/宏观指标构成、方法论原文是什么、某天的值为什么是这个数。当用户询问某因子的构成、依赖、计算逻辑、方法论，或"这个因子为什么是这个值"时使用。基于 Betalpha Mesh 的 get_factor_lineage 与 explain_factor_value 工具完成，不写代码。资产买卖信号的解读请用 betalpha-asset-signal，不要用本技能。
---

# 因子血缘查询与解释

回答两类问题，各走一条固定路径，全程只调 MCP 工具、不写代码、不自行重算：

- **只问"怎么算的/由什么构成"** → `get_factor_lineage`（方法论 + 依赖图）；
- **问"为什么是这个值"** → `explain_factor_value`（值 + 构成 + 缺失原因）。

## 第 1 步：解析因子

用户给的可能是中文名、Provider code 或 `app_bm_` 开头的 canonical code。调用
`search_factors`（免 Key）按名称/代码搜索；命中多个时列出候选让用户确认，
不要猜。确认后可用 `get_factor_metadata`（免 Key）核对 canonical code，
并取 `end_date` 备用。

## 第 2 步：确定数据基准日（T-1，值解释类必做）

系统是 T-1 数据：最新值落在最近一个已同步数据的交易日（即第 1 步拿到的
`end_date`），不在提问当天。问"为什么是这个值"时，后续查询显式传
`date=end_date`；只问方法论则不需要日期。回答中的数据时点一律以返回的
`as_of_date` 为准。

## 第 3 步：按问题类型查询

**A. 方法论/构成**：调用 `get_factor_lineage`（`code`）。读两处：

- `lineage.methodology_doc`：方法论原文（description / implementationLogic /
  beifaPerspective）；
- `lineage.references[]`：直接依赖，按 `role` 分组转述——
  - `role=weight`：根因子按这些权重加权上游信号；
  - `role=source`：数据来源或上游输入；
  - `role=member`：成员关系；
  - `asset_type=unknown` 的引用是**外部节点**（组合、标签等，本地目录无法解析），
    按"code 已知、类型未知"如实转述，不要编造其含义。

**B. 值解释**：调用 `explain_factor_value`（`code` + `date=基准日`，
`asset_codes` 可选）。要点：

- `nodes[0]` 是根因子；默认只展开直接依赖（depth 1），确需更深时显式传
  `depth`（范围 1..=3，再大也会被服务端钳制到 3，层数越多响应越大）；
- 每个节点带 `values`（`source=cache|provider`）、`missing`（稳定原因码）、
  可选 `error`；缺失原因**原样转述**，不猜测；
- `warnings` 出现 `LINEAGE_METADATA_ONLY` 时，必须告知用户：该因子还没有
  人工确认的血缘图，返回的是元数据兜底，不要据此推断隐藏依赖。

## 第 4 步：按固定结构回答

1. **一句话方法论**：从 `methodology_doc` 提炼计算逻辑；
2. **直接依赖**：按 `role` 分组列出名称 + 代码，外部节点单独标注；
3. **值解释**（如适用）：根因子值、关键依赖的值/缺失原因、`as_of_date`；
4. **边界声明**：血缘是方案 A 描述性文档，不承诺精确复算；数据截至
   `as_of_date`（T-1）。

## 规则（必须遵守）

- 血缘是**描述性**的（`recomputable=false` 常态）：只转述方法论与构成，
  不自行重算加权和、不用依赖值反推根值；
- `get_factor_lineage` / `explain_factor_value` 需要 API Key、消耗配额；
  `search_factors` / `get_factor_metadata` 免 Key，优先用它们缩小范围；
- 缺失、外部节点、`LINEAGE_METADATA_ONLY` 一律如实报告；
- 全程不写代码。
