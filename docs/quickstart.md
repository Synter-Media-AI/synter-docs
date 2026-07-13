# Quick Start Guide

Two ways to build on Synter: the **MCP server** for agent-driven use (recommended), and the **REST API** for deterministic server-to-server automation. Pick the one that fits.

## Prerequisites

- A Synter account. Sign up at [syntermedia.ai](https://syntermedia.ai).
- Your ad accounts connected (Settings, then Connections).
- An API key that starts with `syn_`. Create one at [syntermedia.ai/developer](https://syntermedia.ai/developer).

The same `syn_` key works for both the MCP and the REST API.

## Path 1: The MCP server (recommended)

This is how most people use Synter programmatically. Your AI client (Claude, Cursor, Codex, and others) gets tools to read and manage advertising in natural language.

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

Then just ask: "Pull my Google Ads performance for the last 30 days" or "Pause any campaign spending over $100/day with ROAS below 2." Write actions ask for your approval before anything spends money.

Full setup for every client, plus the packaged plugin (skills, agents, safety hook), is in the [Claude Plugin & MCP guide](./guides/claude-plugin.md).

## Path 2: The REST API

Use this for backend jobs, webhooks, and scheduled automation where there is no agent in the loop. One endpoint runs any Synter tool.

- **Base URL:** `https://syntermedia.ai/api/v1`
- **Auth:** `Authorization: Bearer syn_...` (the REST API does not accept `X-Synter-Key`; that is MCP-only)

### List the tools your key can run

```bash
curl https://syntermedia.ai/api/v1/tools/run \
  -H "Authorization: Bearer $SYNTER_API_KEY"
```

### Run a tool

`POST /api/v1/tools/run` with a JSON body:

| Field | Required | Description |
|-------|----------|-------------|
| `script_name` | yes | The tool to run (lowercase, underscores). See the list endpoint above. |
| `args` | no | Array of string CLI-style arguments for the tool. |
| `platform` | no | Platform hint (`google`, `meta`, `reddit`, ...). |
| `customer_id` | no | The ad account ID to scope the action to. Must be connected in this key's workspace. |

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

Remove `--dry-run` to actually upload. See the [Conversion Tracking guide](./guides/conversion-tracking.md) for the full offline-conversion workflow.

### Scopes, credits, and errors

- Reads (`pull_*` and other read tools) require the `tools:read` scope. Writes require `tools:write`.
- Write tools deduct credits up front and refund automatically if the tool fails.
- Common responses: `401` (missing or invalid key), `402` (`INSUFFICIENT_CREDITS`), `403` (missing scope, or `UPGRADE_REQUIRED` on a read-only plan), `400` (`UNKNOWN_TOOL` or a validation error), `502` (backend error, credits refunded).

## Next Steps

- [REST API from Node](../sdk/typescript/README.md)
- [REST API from Python](../sdk/python/README.md)
- [Conversion Tracking Guide](./guides/conversion-tracking.md)
- [AI Agents Guide](./guides/ai-agents.md)
- [Claude Plugin & MCP](./guides/claude-plugin.md)
