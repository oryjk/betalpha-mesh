# betalpha-mesh-skills · Betalpha Mesh 投研技能集

中文 | [English](README.en.md)

面向 AI agent（ZCode / Claude Code / Cursor 等）的 Betalpha Mesh 场景技能集，
把"邀请码注册 → MCP 接入 → 投研提问"整条链路固化成确定性流程。安装后无需写任何
代码，agent 按技能内置的编排调用 Betalpha Mesh 的 MCP 工具完成取数与解读。

## 技能清单

| 技能 | 触发场景 |
| --- | --- |
| `betalpha-mesh-onboarding` | 提供邀请码注册账号、配置 MCP 客户端连接 |
| `betalpha-asset-signal` | 问某只基金/股票/ETF 的买卖信号、多空方向、"该买还是该卖" |
| `betalpha-factor-lineage` | 问某因子怎么计算、由什么构成、方法论是什么、为什么是这个值 |

## 安装

任何兼容 plugin marketplace 的客户端（以 ZCode 为例）：

```
/plugin marketplace add oryjk/betalpha-mesh
/plugin install betalpha-mesh-skills@betalpha-mesh
```

手动安装（可选）：把本仓库 `skills/` 下的各技能目录复制到 `~/.agents/skills/`。

仓库内同时提供了 `.zcode-plugin`、`.claude-plugin`、`.codex-plugin`、
`.cursor-plugin`、`.devin-plugin`、`.hermes-plugin`、`.kimi-plugin`、`.opencode`、
`.pi/extensions`、`.agents/plugins` 各客户端约定的清单目录（内容为同一份
plugin.json），按你的客户端对应的插件机制安装即可。

仓库根目录的 [SKILL.md](SKILL.md) 是技能集路由入口：若你的工具按"单技能仓库"
方式导入（要求根目录有 SKILL.md），导入后它会按问题类型把调用路由到上述三个
子技能。

## 从邀请码开始：三步上手

### 前置

- Betalpha Mesh 服务地址：`https://mesh.betalpha.com`；
- 一个 `bmi_` 开头的一次性邀请码，由 Betalpha Mesh 管理员签发。

### 第 1 步：注册拿 Key

安装技能集后，直接对 agent 说：

> 我有一个 Betalpha Mesh 邀请码 bmi_xxx…，帮我注册并配置连接

`betalpha-mesh-onboarding` 技能会接手：调用注册端点、拿到**仅出现一次**的明文
API Key（`bm_` 开头），并按你的客户端类型渲染好配置。

不用技能也可以手动注册：

```bash
curl -sS -X POST "https://mesh.betalpha.com/api/v1/auth/register-with-invitation" \
  -H "Content-Type: application/json" \
  -d '{"invitation_code":"bmi_xxxxxxxx"}'
```

响应 `data.api_key` 即明文 Key，请立即存入密码管理器——服务端无法再次提供。

### 第 2 步：配置 MCP 连接

在 MCP 客户端中添加 Streamable HTTP 服务，端点 `https://mesh.betalpha.com/mcp`，
请求头携带 `X-API-Key: {API_KEY}`。以 Claude Code 为例：

```bash
claude mcp add --transport http betalpha-mesh https://mesh.betalpha.com/mcp \
  --header "X-API-Key: {API_KEY}"
```

Claude Desktop / Cursor 的 JSON 配置：

```json
{
  "mcpServers": {
    "betalpha-mesh": {
      "type": "http",
      "url": "https://mesh.betalpha.com/mcp",
      "headers": { "X-API-Key": "{API_KEY}" }
    }
  }
}
```

常见连接问题（Host 白名单、Origin 放行、`INVALID_API_KEY`、`QUOTA_EXCEEDED`）
见 onboarding 技能的 `references/client-configs.md`。

### 第 3 步：开始提问

配置重连后验证一次：免 Key 调用 `search_factors`（连通性），再调用
`get_factor_snapshot`（鉴权与配额）。然后就可以直接用自然语言提问：

- **买卖信号**：「华夏成长（000001）的红利买卖信号是多少？现在该买还是该卖？」
  → `betalpha-asset-signal`：解析资产代码（000001 有股/基金歧义时会先和你确认），
  按 T-1 基准日取最新信号与近一月序列，给出方向、相对位置、构成拆解和方法论。
- **因子血缘**：「买卖信号_红利基金这个因子是怎么算出来的？由什么构成？」
  → `betalpha-factor-lineage`：方法论、直接依赖（按 weight/source 分组）、
  或到指定日期的值级解释。
- **自由探索**（无需技能，免 Key 不耗配额）：`search_factors`、
  `get_factor_metadata`、`search_assets`、`search_macro_indicators` 直接问 agent
  "有哪些因子/资产/宏观指标"即可。

## 使用须知

- **数据是 T-1 的**：一切"最新"指最近一个已同步数据的交易日，不是提问当天；
  技能的回答都会显式携带 `as_of_date`。
- **信号是标准化相对分**：只描述方向与相对位置，不是投资建议；技能会拒绝把
  数值换算成"强烈买入/卖出"。
- **配额与计量**：快照、时序、血缘、解释类工具消耗每日配额（档位由管理员分配）；
  搜索与元数据类工具免 Key 免配额。
- **安全**：明文 Key 只在注册响应出现一次，不写入日志、笔记或第三方服务。

## 反馈

本仓库是 betalpha-mesh-skills 的发布产物，请勿直接在此提交修改；问题或建议请
通过 issue 反馈。

## 版本

- 0.1.0：betalpha-mesh-onboarding、betalpha-asset-signal、betalpha-factor-lineage

## License

MIT License - see [LICENSE](LICENSE) file for details.
