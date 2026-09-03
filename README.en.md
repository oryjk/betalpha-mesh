# betalpha-mesh-skills · Betalpha Mesh Research Skills

[中文](README.md) | English

Scenario skills for AI agents (ZCode / Claude Code / Cursor, etc.) working with
Betalpha Mesh. They turn the full "invitation registration → MCP setup → research
questions" journey into deterministic workflows. Once installed, the agent needs
zero hand-written code: it follows each skill's built-in orchestration to call
the MCP tools of Betalpha Mesh for data retrieval and interpretation.

## Skills

| Skill | Use when |
| --- | --- |
| `betalpha-mesh-onboarding` | Registering an account with an invitation code and configuring MCP clients |
| `betalpha-asset-signal` | Asking about the buy/sell signal or long/short direction of a fund/stock/ETF, or "should I buy or sell now" |
| `betalpha-factor-lineage` | Asking how a factor is computed, what it is composed of, its methodology, or why it has a certain value |

## Installation

Any client with a compatible plugin marketplace (ZCode for example):

```
/plugin marketplace add oryjk/betalpha-mesh
/plugin install betalpha-mesh-skills@betalpha-mesh
```

Manual install (optional): copy each skill directory under `skills/` into
`~/.agents/skills/`.

The repository also ships manifest directories for a range of clients —
`.zcode-plugin`, `.claude-plugin`, `.codex-plugin`, `.cursor-plugin`,
`.devin-plugin`, `.hermes-plugin`, `.kimi-plugin`, `.opencode`, `.pi/extensions`,
and `.agents/plugins` (all containing the same plugin.json) — so install with
whichever plugin mechanism your client supports.

The root [SKILL.md](SKILL.md) is the collection's routing entry: if your tool
imports repositories as a single skill (requiring a root SKILL.md), it routes
each question to one of the three skills above.

## Getting started: three steps from an invitation code

### Prerequisites

- The Betalpha Mesh service address: `https://mesh.betalpha.com`;
- A one-time invitation code prefixed with `bmi_`, issued by a Betalpha Mesh admin.

### Step 1: Register and get an API key

With the skills installed, just tell the agent:

> I have a Betalpha Mesh invitation code bmi_xxx…, register me and set up the
> connection

The `betalpha-mesh-onboarding` skill takes over: it calls the registration
endpoint, obtains the API key (starting with `bm_`) — **shown exactly once** —
and renders the config for your client type.

You can also register manually without the skill:

```bash
curl -sS -X POST "https://mesh.betalpha.com/api/v1/auth/register-with-invitation" \
  -H "Content-Type: application/json" \
  -d '{"invitation_code":"bmi_xxxxxxxx"}'
```

`data.api_key` in the response is the plaintext key. Store it in a password
manager immediately — the server can never show it again.

### Step 2: Configure the MCP connection

Add a Streamable HTTP MCP server in your client: endpoint
`https://mesh.betalpha.com/mcp` with an `X-API-Key: {API_KEY}` request header.
For Claude Code:

```bash
claude mcp add --transport http betalpha-mesh https://mesh.betalpha.com/mcp \
  --header "X-API-Key: {API_KEY}"
```

JSON config for Claude Desktop / Cursor:

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

For common connection issues (Host allowlist, Origin requirements,
`INVALID_API_KEY`, `QUOTA_EXCEEDED`) see `references/client-configs.md` in the
onboarding skill.

### Step 3: Start asking

After reconnecting, verify once: call `search_factors` (keyless, connectivity)
and `get_factor_snapshot` (auth and quota). Then ask in plain language:

- **Buy/sell signals**: "What is the dividend buy/sell signal of Huaxia Growth
  (000001)? Should I buy or sell now?" → `betalpha-asset-signal`: resolves the
  asset code (ambiguous codes like 000001 are confirmed with you first), reads
  the latest signal and one-month history as of the T-1 base date, then reports
  direction, relative position, composition breakdown, and methodology.
- **Factor lineage**: "How is the fund dividend traffic signal factor computed?
  What is it composed of?" → `betalpha-factor-lineage`: methodology, direct
  dependencies (grouped by weight/source), or a value-level explanation for a
  given date.
- **Free exploration** (no skill needed; keyless and quota-free): ask the agent
  "what factors/assets/macro indicators are available" using `search_factors`,
  `get_factor_metadata`, `search_assets`, and `search_macro_indicators`.

## Notes

- **Data is T-1**: every "latest" value refers to the most recent synced
  trading day, not the moment you ask; skill answers always carry `as_of_date`.
- **Signals are standardized relative scores**: they describe direction and
  relative position only, and are not investment advice; the skills refuse to
  translate raw values into "strong buy/sell" language.
- **Quota and metering**: snapshot, series, lineage, and explanation tools
  consume your daily quota (tier assigned by admins); search and metadata tools
  are keyless and quota-free.
- **Security**: the plaintext key appears exactly once in the registration
  response; never write it into logs, notes, or third-party services.

## Feedback

This repository is the release artifact of betalpha-mesh-skills; please do not
submit changes here directly. Open an issue for problems or suggestions.

## Version

- 0.1.0: betalpha-mesh-onboarding, betalpha-asset-signal, betalpha-factor-lineage

## License

MIT License - see the [LICENSE](LICENSE) file for details.
