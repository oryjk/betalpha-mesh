---
name: "betalpha-active-fund-star-rating"
description: "输入任意一只主动权益基金代码，查询其主动基金综合评估标签取值并映射为一至五星评级。当用户询问某只主动权益基金的评估结果、星级或综合评估时触发。"
---

# 主动权益基金星级评估查询

场景意图：用户给定一只主动权益基金，输出其星级评估。日期规则：未指定日期时 get_factor_snapshot 默认取最新开放交易日，并在结果中展示 as_of_date。步骤：先用 search_assets 在 fund 资产类型下按用户输入消歧得到 asset_code；再对根因子 app_bm_fund_activefund_comprehensive_assessment_qualitative 查询单基金截面，读取 value（离散值 1–5），按 rules 中的星级映射输出，不自行复算综合评估得分或分位。缺失处理：若该基金出现在 missing 列表或值为空，则报告无法评估，可能原因包括非主动权益基金（Tag_f_active_equityA 取值为 0）或 QDII 基金，但不猜测具体原因；不查询其子标签。输出结构：基金名称/代码、评估日期（as_of_date）、标签原始值、星级结论、缺失原因（如有）。追问分支：若用户追问星级如何得来，可调用 get_factor_lineage 展示已发布血缘说明（分组分位映射），仍不重新计算。

## MCP 调用步骤

### search1 · search_assets

执行条件：用户查询某只基金的星级评估

按用户输入的基金代码或名称在 fund 资产类型下搜索消歧，得到 asset_code；结果为空或不唯一时向用户澄清

工具：`search_assets`

```json
{
  "asset_type": "fund",
  "query": {
    "$input": "query"
  }
}
```

### snap1 · get_factor_snapshot

执行条件：asset_code 已确定

读取该基金在指定日期（缺省最新交易日）的标签值；value 为离散值 1-5，交由客户 Agent 按 rules 映射为星级；若资产在 missing 中，展示 reasons 并报告不可评估

工具：`get_factor_snapshot`

```json
{
  "asset_codes": [
    {
      "$from": "search1#/items/0/asset_code"
    }
  ],
  "code": "app_bm_fund_activefund_comprehensive_assessment_qualitative",
  "date": {
    "$input": "date"
  }
}
```

### lineage1 · get_factor_lineage

执行条件：用户追问星级评估的计算依据

仅当用户追问评估来源时调用，展示已发布血缘与 methodology_doc，用于解释，不重新计算

工具：`get_factor_lineage`

```json
{
  "asset_type": "fund",
  "code": "app_bm_fund_activefund_comprehensive_assessment_qualitative"
}
```

`$from` 表示使用前序响应的 JSON Pointer 字段，`$input` 表示用户参数，调用时必须替换为实际值。字段或日期缺失时报告缺失，不传引用对象，不猜测值。游标按工具返回原样续取。无效 Key 或配额耗尽时停止受保护调用。

## 结果解释规则

以下规则由使用本 Skill 的 Agent 在取得 MCP 真实返回值后执行；不得猜测缺失数据。

### star-mapping

根据 snap1 返回的 value（离散整数 1-5）映射星级并输出：5→五星，4→四星，3→三星，2→二星，1→一星。不得把离散档位扩展为连续区间，也不得改用综合评估得分自行重算分位。value 缺失或不在 1-5 枚举内时报告无法评估。

原文依据：如果标签取值为5，则基金评估为“五星”
如果标签取值为4，则基金评估为“四星”
如果标签取值为3，则基金评估为“三星”
如果标签取值为2，则基金评估为“二星”
如果标签取值为1，则基金评估为“一星”

缺失或未定义值处理：value 缺失、为 null 或不在 1-5 枚举内时，输出无法评估，并原样展示 missing.reasons；不猜测星级，不使用子因子重算。

### coverage-scope

向用户说明适用范围：仅主动权益基金有评估结果；该范围说明来自文档，未在本次查询中逐基金验证分类标签。

原文依据：主动权益基金是主动权益基金标签（基金标签Code：Tag_f_active_equityA）取值为1的基金；该标签取值为0的基金没有评估。该类别中的QDII基金也没有评估。

缺失或未定义值处理：查询结果为空或缺失时，提示该基金可能不属于评估覆盖范围（非主动权益基金或 QDII），但不据此断言其分类。
