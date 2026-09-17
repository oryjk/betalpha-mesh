---
name: "betalpha-dividend-style-rotation"
description: "当用户查询核心红利、类债红利、周期红利的买卖信号或追问组合股票持仓时，查询真实宏观指标，按场景阈值判定，并获取最近交易日持仓及权重。"
---

# 红利风格轮动信号

当用户问红利风格轮动、核心/类债/周期红利买卖信号时，查询三类宏观信号，各自使用最新可用日期。按用户原文阈值：>1 买入；<-0.5 卖出；[-0.5,1] 谨慎。保留原始数值精度，缺失不判谨慎。展示风格、实际观测日期、值及场景标签；日期不同不做同日排名，说明日期差异。用户问指定历史日期时，把各值步骤 date 替换成其明确日期，不使用最新日期。用户追问某组合持仓才执行对应 holdings 步骤；未指明组合时先澄清。持仓分数权重转百分比展示，不改变原值。Key 无效/配额耗尽停止受保护调用；服务不可用、无数据均如实报告，不补造结果。阈值仅为本文场景规则，不推广到其他指标。当前三个根指标无已发布血缘，禁止声称获得完整因果链或自行计算因子。

## MCP 调用步骤

### core_metadata · get_macro_indicator_metadata

执行条件：用户查询红利风格信号

获取核心红利最新可用观测日期 end_date；空日期则该类不可判定，跳过值查询。

工具：`get_macro_indicator_metadata`

```json
{
  "code": "app_bm_macro_port_core_dividend_signal_3factor_tzscore"
}
```

### core_value · get_macro_value

执行条件：对应元数据 end_date 存在

读取核心红利的 value.value，以 dividend_signal 规则判定；null 为不可判定，展示实际 value.date，不将请求日期冒充观测日期。

工具：`get_macro_value`

```json
{
  "code": "app_bm_macro_port_core_dividend_signal_3factor_tzscore",
  "date": {
    "$from": "core_metadata#/end_date"
  }
}
```

### core_holdings · get_portfolio_holdings

执行条件：用户点选或追问核心红利的持仓

省略 date 取北京时间今天之前最近交易日；用户指定日期时传用户日期，非交易日应请用户改选。按 next_cursor 逐页取完，date 固定为首个响应 as_of_date；返回 asset_type=stock 的股票代码、名称和原始 weight，百分比展示乘100但不重新归一化，其他/未知资产单列。解释 date_basis=provider_query_date 表示回源请求日，源未提供独立持仓观测日，不声称已验证持仓当日更新。空持仓明确无数据，不从血缘推断。

工具：`get_portfolio_holdings`

```json
{
  "portfolio_code": "port_core_dividend",
  "limit": 100
}
```

### debt_metadata · get_macro_indicator_metadata

执行条件：用户查询红利风格信号

获取类债红利最新可用观测日期 end_date；空日期则该类不可判定，跳过值查询。

工具：`get_macro_indicator_metadata`

```json
{
  "code": "app_bm_macro_port_debt_dividend_signal_2factor_tzscore"
}
```

### debt_value · get_macro_value

执行条件：对应元数据 end_date 存在

读取类债红利的 value.value，以 dividend_signal 规则判定；null 为不可判定，展示实际 value.date，不将请求日期冒充观测日期。

工具：`get_macro_value`

```json
{
  "code": "app_bm_macro_port_debt_dividend_signal_2factor_tzscore",
  "date": {
    "$from": "debt_metadata#/end_date"
  }
}
```

### debt_holdings · get_portfolio_holdings

执行条件：用户点选或追问类债红利的持仓

省略 date 取北京时间今天之前最近交易日；用户指定日期时传用户日期，非交易日应请用户改选。按 next_cursor 逐页取完，date 固定为首个响应 as_of_date；返回 asset_type=stock 的股票代码、名称和原始 weight，百分比展示乘100但不重新归一化，其他/未知资产单列。解释 date_basis=provider_query_date 表示回源请求日，源未提供独立持仓观测日，不声称已验证持仓当日更新。空持仓明确无数据，不从血缘推断。

工具：`get_portfolio_holdings`

```json
{
  "portfolio_code": "port_debt_dividend",
  "limit": 100
}
```

### period_metadata · get_macro_indicator_metadata

执行条件：用户查询红利风格信号

获取周期红利最新可用观测日期 end_date；空日期则该类不可判定，跳过值查询。

工具：`get_macro_indicator_metadata`

```json
{
  "code": "app_bm_macro_port_period_dividend_signal_4factor_tzscore"
}
```

### period_value · get_macro_value

执行条件：对应元数据 end_date 存在

读取周期红利的 value.value，以 dividend_signal 规则判定；null 为不可判定，展示实际 value.date，不将请求日期冒充观测日期。

工具：`get_macro_value`

```json
{
  "code": "app_bm_macro_port_period_dividend_signal_4factor_tzscore",
  "date": {
    "$from": "period_metadata#/end_date"
  }
}
```

### period_holdings · get_portfolio_holdings

执行条件：用户点选或追问周期红利的持仓

省略 date 取北京时间今天之前最近交易日；用户指定日期时传用户日期，非交易日应请用户改选。按 next_cursor 逐页取完，date 固定为首个响应 as_of_date；返回 asset_type=stock 的股票代码、名称和原始 weight，百分比展示乘100但不重新归一化，其他/未知资产单列。解释 date_basis=provider_query_date 表示回源请求日，源未提供独立持仓观测日，不声称已验证持仓当日更新。空持仓明确无数据，不从血缘推断。

工具：`get_portfolio_holdings`

```json
{
  "portfolio_code": "port_period_dividend",
  "limit": 100
}
```

`$from` 表示前序响应的 JSON Pointer 字段；调用时替换为真实标量，不把引用对象传给 MCP。字段缺失时跳过并报告。

## 场景判定规则

使用原始精度比较；null 为不可判定。

```json
[
  {
    "id": "dividend_signal",
    "bands": [
      {
        "label": "卖出",
        "min": null,
        "max": -0.5,
        "min_inclusive": false,
        "max_inclusive": false
      },
      {
        "label": "谨慎",
        "min": -0.5,
        "max": 1,
        "min_inclusive": true,
        "max_inclusive": true
      },
      {
        "label": "买入",
        "min": 1,
        "max": null,
        "min_inclusive": false,
        "max_inclusive": false
      }
    ]
  }
]
```
