---
name: betalpha-mesh-onboarding
description: 使用一次性邀请码为用户注册 Betalpha Mesh 账号，获取首个 API Key，并生成各 MCP 客户端（Claude Code、Claude Desktop、Cursor 等）可直接粘贴的连接配置。当用户提供 bmi_ 开头的邀请码、想注册或接入 Betalpha Mesh、或需要配置 Betalpha Mesh MCP 服务时使用。
---

# Betalpha Mesh 接入引导

把用户的一次性邀请码变成一份可直接粘贴的 MCP 客户端配置。你只做三件事：
调用 REST 注册端点、渲染配置文本、引导验证连接。不要发明其他步骤。

## 第 1 步：收集输入

向用户确认以下信息（服务地址有默认值，其余两项缺一不可，不要猜测）：

1. **邀请码**：`bmi_` 前缀 + 43 位随机字符串，由 Betalpha Mesh 管理员签发。
2. **服务地址**：默认 `https://mesh.betalpha.com`（官方服务，MCP 与 REST 同进程
   同端口）；仅当用户明确提供其他 Betalpha Mesh 服务地址时才改用。
3. **目标 MCP 客户端**：Claude Code、Claude Desktop、Cursor，还是其他 Streamable HTTP 客户端。

## 第 2 步：调用 REST 注册

```
curl -sS -X POST "{服务地址}/api/v1/auth/register-with-invitation" \
  -H "Content-Type: application/json" \
  -d '{"invitation_code":"<用户提供的邀请码>"}'
```

该端点无需任何鉴权。响应为统一封装：

- 成功（HTTP 201）：`{"success":true,"data":{...}}`，`data` 包含
  `account_id`、`api_key`（**完整明文，仅此一次**）、`key_prefix`、`last_four`、
  `name`、`tier`、`expires_at`。
- 失败（HTTP 400）：`{"success":false,"error":{"code":"INVALID_INVITATION_CODE",...}}`。

## 第 3 步：处理结果

**成功时**：

- 向用户复述非敏感元数据：`key_prefix`、`last_four`、`tier`、`expires_at`；
- 立即进入第 4 步渲染配置，不要先停下来问"要继续吗"。

**失败时**：

- `INVALID_INVITATION_CODE`：告知用户邀请码无效、已过期、已被使用或已撤销（服务端
  不区分具体状态），需要中台管理员重新签发。**不要**建议重试同一个码或猜测格式。
- 连接被拒 / 超时：提示检查服务地址是否可达。

## 第 4 步：渲染 MCP 客户端配置

按用户客户端类型，从 [references/client-configs.md](references/client-configs.md)
取对应模板，填入服务地址与 `data.api_key`，把完整配置输出给用户，让用户粘贴并重连。

同时必须告知用户：

1. 明文 API Key 仅在本次注册响应中出现一次，服务端无法再次提供，建议立即存入密码管理器；
2. 粘贴配置并重连后，回来继续第 5 步验证。

## 第 5 步：验证连接

用户重连 MCP 后，依次建议两个验证：

1. 连通性：调用免 Key 工具 `search_factors`（参数 `query` 任选，如 `"yield"`）；
2. 鉴权与配额：调用需 Key 工具 `get_factor_snapshot`（任选一个因子代码），确认不再返回
   `INVALID_API_KEY`。

验证通过后，主动引导用户体验场景技能（若已安装 betalpha-mesh-skills 技能集）：

3. 让用户试试询问某资产的买卖信号，例如**"华夏成长（000001）的红利买卖信号是多少？
   现在该买还是该卖？"**——这句话会触发 `betalpha-asset-signal` 技能，它按固定编排
   完成取数与解读。技能未安装时，告知用户安装 betalpha-mesh-skills 技能集可获得
   场景化解读能力。

**连接被拒时**优先排查 Host 白名单：客户端 URL 中的主机名必须在服务端 `MCP_ALLOWED_HOSTS`
内，否则会被 DNS rebinding 防护拒绝。把 URL 主机名报给 Mesh 管理员即可。

## 安全规则（必须遵守）

- 邀请码和明文 Key **不写入**任何日志、临时文件、版本库、笔记或外部服务；
- 明文 Key 只出现在第 4 步输出的配置文本里，不重复展示，不进入你的持久化记忆；
- 不要把邀请码或 Key 粘贴到任何第三方工具或在线服务；
- 注册事务在服务端原子完成：并发重复兑换只有一个会成功，无需客户端去重。
