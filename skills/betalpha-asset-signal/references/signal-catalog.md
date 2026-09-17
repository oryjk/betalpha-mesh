# 信号因子目录

按资产类型列出当前已上架的买卖信号因子。新增信号因子时在此登记。
主流程（SKILL.md 第 2 步）按 `asset_type` 匹配本目录；目录未覆盖时用
`search_factors(query="signal")` 兜底。

## fund（基金）

### app_bm_fund_betalpha_dividend_traffic_signal · 买卖信号_红利基金

- **覆盖**：与倍发三类红利组合有持仓暴露的主动权益基金；其余基金 `VALUE_NOT_FOUND`。
- **值形态**：连续值，日频。三个红利组合标准化买卖信号按基金拆解权重加权，
  实际量级 ~1e-5；>0 偏多，<0 偏空。
- **构成**（`explain_factor_value` 一层血缘）：
  - `role=source` 上游信号：
    `app_bm_macro_port_core_dividend_signal_3factor_tzscore`（核心红利，估值三因子：股息率/FCF 收益率/盈利收益率）；
    `app_bm_macro_port_debt_dividend_signal_2factor_tzscore`（类债红利，股息率/股息利差）；
    `app_bm_macro_port_period_dividend_signal_4factor_tzscore`（周期红利，商品动量/RSI20/盈利预期调整/股息PB比）。
  - `role=weight` 拆解权重（季度，基于持仓重合度排名归一）：
    `app_bm_fund_port_core_dividend_weight_overlap_quarterly`、
    `app_bm_fund_port_debt_dividend_weight_overlap_quarterly`、
    `app_bm_fund_port_period_dividend_weight_overlap_quarterly`。
- **一句话方法论**：基金的买卖信号 = 其在倍发核心/类债/周期三类红利组合上的拆解
  权重 × 各组合自身买卖信号之和；拆解基于季度持仓重合度排名，目标是相对精确而非
  绝对精确。

## stock / etf / index

当前没有已上架的信号因子。遇到此类询问时如实告知，不要用其他因子冒充。
