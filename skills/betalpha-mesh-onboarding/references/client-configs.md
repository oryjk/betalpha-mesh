# MCP 客户端配置模板

按用户客户端类型选用。`{BASE}` 为 Betalpha Mesh 后端地址（含端口，无末尾斜杠），
`{API_KEY}` 为注册响应 `data.api_key` 的完整明文。MCP 与 REST 同进程同端口，路径固定为 `/mcp`。

## Claude Code（CLI）

命令行添加（推荐，写入用户级配置）：

```bash
claude mcp add --transport http betalpha-mesh {BASE}/mcp \
  --header "X-API-Key: {API_KEY}"
```

验证：`claude mcp list` 应显示 betalpha-mesh 且可连接。

## Claude Desktop

`claude_desktop_config.json` 的 `mcpServers` 段：

```json
{
  "mcpServers": {
    "betalpha-mesh": {
      "type": "http",
      "url": "{BASE}/mcp",
      "headers": { "X-API-Key": "{API_KEY}" }
    }
  }
}
```

macOS 配置文件路径：`~/Library/Application Support/Claude/claude_desktop_config.json`。
修改后需重启 Claude Desktop。

## Cursor

项目级 `.cursor/mcp.json` 或全局配置的 `mcpServers` 段，格式与上面相同：

```json
{
  "mcpServers": {
    "betalpha-mesh": {
      "type": "http",
      "url": "{BASE}/mcp",
      "headers": { "X-API-Key": "{API_KEY}" }
    }
  }
}
```

## 通用 Streamable HTTP 客户端

传输：Streamable HTTP（无状态，无服务端 session），端点 `{BASE}/mcp`，
鉴权头 `X-API-Key: {API_KEY}`，请求体上限 4 MiB，JSON 响应优先。
支持 2026-07-28 稳定规范（向后兼容 2025-11-25）。

## 常见问题

- **连接被拒 / 初始化失败**：URL 中的主机名不在服务端 `MCP_ALLOWED_HOSTS` 白名单内
  （DNS rebinding 防护）。把 URL 主机名报给 Mesh 管理员加入白名单，或改用已放行的地址。
- **浏览器客户端额外需要 Origin 放行**：服务端 `MCP_ALLOWED_ORIGINS` 需包含页面 Origin；
  原生（无 Origin 头）客户端不受此限制。
- **`INVALID_API_KEY`（工具级错误，非 HTTP 401）**：Key 未配上或已吊销。确认粘贴的是完整
  `bm_` 明文且无多余空格；Key 明文只在注册响应出现一次，丢失需在服务端重新签发。
- **`QUOTA_EXCEEDED`**：当日配额用尽，属正常计量行为，不是配置错误。
