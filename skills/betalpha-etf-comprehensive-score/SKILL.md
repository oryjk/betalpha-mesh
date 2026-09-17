---
name: "betalpha-etf-comprehensive-score"
description: "输入任意一只权益型ETF代码，查询其ETF综合评估标签值并映射为四星/三星/二星/一星；用于单只ETF综合评估结果的快速查询。"
---

# ETF综合评估星级查询

场景意图：用户给出任意一只权益型ETF代码，查询该ETF的ETF综合评估标签（canonical code：app_bm_fund_etffund_comprehensive_score），并按标签值映射输出星级。

日期规则：用户未指定日期时，get_factor_snapshot 省略 date，取最新开放交易日；用户指定日期时传入该日期。展示实际返回的 as_of_date。

查询方式：使用 get_factor_snapshot，code 固定为 app_bm_fund_etffund_comprehensive_score，asset_codes 传入用户提供的ETF代码（数组）。若用户未提供代码，先追问ETF代码。

结果解释：从 values 中取该 asset_code 的 value，按 rules 中的枚举映射输出星级；若该资产出现在 missing 中，如实报告缺失及 reasons，不推断星级，不猜测缺失原因。

覆盖范围说明（文档说明，非已查询验证事实）：文档称ETF为标签 tag_F_ETFfund_EquityA_Onsite 取值为1的基金，取值为0的基金没有评估，该类别中的QDII基金也没有评估。该标签未纳入本次所选能力，不额外查询或验证；如用户明确要求验证覆盖范围而所选能力无法执行，则说明无法验证。

血缘说明：本场景只查询根指标已计算结果，不重新计算血缘中的子指标（ETF配置属性评估、ETF工具属性评估、场内ETF等）；血缘依赖仅用于解释来源，不逐个绑定或查询。

输出结构：ETF代码与名称（如可得）、评估日期 as_of_date、标签值、对应星级；缺失时输出缺失原因。

追问分支：未提供ETF代码时追问代码；代码无法匹配到资产时报告未返回该资产；用户要求解释评估来源时可说明其由敞口风险特征与可投资性两方面评估构成（文档说明）。

## MCP 调用步骤

### snap1 · get_factor_snapshot

执行条件：用户提供ETF代码并查询综合评估

查询该ETF的ETF综合评估标签值；读取 values 中对应 asset_code 的 value 与 as_of_date；若出现在 missing 中则记录 reasons，不推断星级。

工具：`get_factor_snapshot`

```json
{
  "asset_codes": {
    "$input": "asset_codes"
  },
  "code": "app_bm_fund_etffund_comprehensive_score"
}
```

`$from` 表示使用前序响应的 JSON Pointer 字段，`$input` 表示用户参数，调用时必须替换为实际值。字段或日期缺失时报告缺失，不传引用对象，不猜测值。游标按工具返回原样续取。无效 Key 或配额耗尽时停止受保护调用。

## 结果解释规则

以下规则由使用本 Skill 的 Agent 在取得 MCP 真实返回值后执行；不得猜测缺失数据。

### interpret

根据 get_factor_snapshot 返回的 values 中该 asset_code 的 value 判定星级：value 为 4 输出四星；value 为 3 输出三星；value 为 2 输出二星；value 为 1 输出一星。同时展示实际 as_of_date。

原文依据：如标签值为4，则为四星
如标签值为3，则为三星
如标签值为2，则为二星
如标签值为1，则为一星

缺失或未定义值处理：若该资产出现在 missing 中或 values 中无该资产，如实报告缺失及 reasons，不推断星级，不猜测缺失原因；若 value 为上述枚举之外的值，报告该值并说明无法按文档映射，不自行扩展区间。

### coverage

覆盖范围说明（文档说明，非已查询验证事实）：ETF为标签 tag_F_ETFfund_EquityA_Onsite 取值为1的基金；取值为0的基金没有评估；该类别中的QDII基金也没有评估。该标签未纳入本次所选能力，不额外查询或验证。

原文依据：ETF是ETF标签（基金标签Code：tag_F_ETFfund_EquityA_Onsite）取值为1的基金；该标签取值为0的基金没有评估。该类别中的QDII基金也没有评估。

缺失或未定义值处理：该覆盖范围为文档说明，不作为已查询验证的事实；若用户明确要求验证覆盖范围而所选能力无法执行，说明无法验证，不推断具体原因。
