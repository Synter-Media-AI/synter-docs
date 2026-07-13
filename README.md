# Synter Documentation

**The agent-native platform for multi-platform ad management.**

Manage Google Ads, Meta, LinkedIn, Reddit, Microsoft, TikTok, X, Amazon DSP, The Trade Desk, and more from one place. There are two ways to build on Synter:

1. **The MCP server** (recommended) — give Claude, Cursor, Codex, or any MCP client the ability to read and manage your advertising in natural language. This is how most people use Synter programmatically.
2. **The REST API** — a single authenticated endpoint for deterministic, server-to-server automation (conversion upload, data pulls, scheduled jobs) where you do not want an agent in the loop.

There is no `@synter/sdk` client library and no `pip install synter` package. If you saw those referenced anywhere, they were never published. Use the MCP or the REST API below.

## 📚 Documentation

- [Quick Start Guide](./docs/quickstart.md) - Connect the MCP and make your first REST call
- [Claude Plugin & MCP](./docs/guides/claude-plugin.md) - Run Synter inside Claude Code / Claude Desktop / Cursor / Codex
- [REST API from Node](./sdk/typescript/README.md) - Call the REST API from TypeScript/Node
- [REST API from Python](./sdk/python/README.md) - Call the REST API from Python
- [Conversion Tracking](./docs/guides/conversion-tracking.md) - Upload offline and server-side conversions
- [AI Agents](./docs/guides/ai-agents.md) - How agents run, and the approval-before-spend model
- [UTM Management](./docs/guides/utm-management.md) - Platform-specific UTM macro reference

## 🚀 Option 1: The MCP server (recommended)

Sign up at [syntermedia.ai](https://syntermedia.ai), connect your ad accounts, and copy your API key (starts with `syn_`). Then point your client at the hosted MCP.

**Claude Code:**

```bash
claude mcp add synter \
  --transport http \
  https://mcp.syntermedia.ai \
  --header "X-Synter-Key: YOUR_API_KEY"
```

**Cursor / Windsurf / Claude Desktop (HTTP):**

```json
{
  "mcpServers": {
    "synter": {
      "type": "http",
      "url": "https://mcp.syntermedia.ai",
      "headers": { "X-Synter-Key": "YOUR_API_KEY" }
    }
  }
}
```

See the [Claude Plugin & MCP guide](./docs/guides/claude-plugin.md) for the packaged plugin (skills, agents, safety hook) and every client's setup.

## 🚀 Option 2: The REST API

One authenticated endpoint runs any Synter tool. Use it for backend jobs, webhooks, and scheduled automation where an agent is not in the loop.

- **Base URL:** `https://syntermedia.ai/api/v1`
- **Auth:** `Authorization: Bearer syn_...` — the same `syn_` key. The REST API does not accept the `X-Synter-Key` header; that is MCP-only.

List the tools available to your key:

```bash
curl https://syntermedia.ai/api/v1/tools/run \
  -H "Authorization: Bearer $SYNTER_API_KEY"
```

Run a tool. This example uploads a batch of offline conversions to Google Ads from a CSV, which is the path for conversions your ad platform cannot see (CRM sales, phone quotes, offline closes):

```bash
curl -X POST https://syntermedia.ai/api/v1/tools/run \
  -H "Authorization: Bearer $SYNTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "script_name": "google_ads_upload_offline_conversions",
    "platform": "google",
    "customer_id": "1234567890",
    "args": ["--csv-url", "https://yourapp.com/exports/conversions.csv", "--conversion-name", "Offline Purchase", "--dry-run"]
  }'
```

Reads (`pull_*`, and other read tools) need the `tools:read` scope. Writes need `tools:write` and deduct credits, which are automatically refunded if the tool fails. Full reference: [REST API from Node](./sdk/typescript/README.md).

## ✨ What you can do

- **Cross-platform management**: read and write campaigns across Google, Meta, LinkedIn, Reddit, Microsoft, TikTok, X, Amazon DSP, The Trade Desk
- **Conversion upload**: server-side and offline conversions (Google Ads offline CSV, Reddit CAPI, and more)
- **Performance data**: pull normalized metrics for your own reporting or models
- **Creative generation**: AI images and video
- **Attribution**: a unified view across platforms, including offline sources

## 🔗 Links

- [Website](https://syntermedia.ai)
- [Developer portal (create an API key)](https://syntermedia.ai/developer)
- [Dashboard](https://syntermedia.ai/dashboard)
- [MCP server on npm](https://www.npmjs.com/package/@synterai/mcp-server)

## 📄 License

MIT
