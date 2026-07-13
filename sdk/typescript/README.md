# Synter REST API from Node/TypeScript

There is no published `@synter/sdk` npm package. Synter's programmatic surface is one authenticated REST endpoint that runs any Synter tool. You call it with `fetch` (built into Node 18+). For agent-driven use in Claude, Cursor, or Codex, use the [MCP server](../../docs/guides/claude-plugin.md) instead.

## Setup

- **Base URL:** `https://syntermedia.ai/api/v1`
- **Auth:** `Authorization: Bearer syn_...` (create a key at [syntermedia.ai/developer](https://syntermedia.ai/developer))

A tiny wrapper is all you need:

```typescript
const BASE = "https://syntermedia.ai/api/v1";

async function runTool(params: {
  script_name: string;
  args?: string[];
  platform?: string;
  customer_id?: string;
}) {
  const res = await fetch(`${BASE}/tools/run`, {
    method: "POST",
    headers: {
      Authorization: `Bearer ${process.env.SYNTER_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify(params),
  });
  const data = await res.json();
  if (!res.ok) {
    throw new Error(`Synter ${res.status}: ${JSON.stringify(data)}`);
  }
  return data;
}
```

## List available tools

```typescript
const res = await fetch("https://syntermedia.ai/api/v1/tools/run", {
  headers: { Authorization: `Bearer ${process.env.SYNTER_API_KEY}` },
});
const { tools } = await res.json();
console.log(tools);
```

Requires the `tools:read` scope. The list reflects the tools your key's plan and workspace can run.

## Run a tool

`POST /api/v1/tools/run`:

| Field | Required | Description |
|-------|----------|-------------|
| `script_name` | yes | Tool name (lowercase, underscores only). |
| `args` | no | Array of string CLI-style arguments. |
| `platform` | no | `google`, `meta`, `linkedin`, `reddit`, `microsoft`, `tiktok`, `x`, ... |
| `customer_id` | no | Ad account ID; must be connected in this key's workspace. |

```typescript
// Pull recent performance (read; needs tools:read)
const perf = await runTool({
  script_name: "pull_google_ads_performance",
  platform: "google",
  customer_id: "1234567890",
  args: ["--days", "30"],
});

// Upload offline conversions from a CSV (write; needs tools:write, deducts credits)
const upload = await runTool({
  script_name: "google_ads_upload_offline_conversions",
  platform: "google",
  customer_id: "1234567890",
  args: [
    "--csv-url", "https://yourapp.com/exports/conversions.csv",
    "--conversion-name", "Offline Purchase",
    // include --dry-run first to validate without uploading
  ],
});
```

Run the exact tool names from the list endpoint. Passing an unknown `script_name` returns `400 UNKNOWN_TOOL` with zero credits charged.

## Scopes, credits, and errors

- **Scopes:** reads need `tools:read`; writes need `tools:write`. A key without the scope gets `403`.
- **Credits:** write tools deduct credits up front and refund automatically if execution fails.
- **Status codes:**
  - `401` missing or invalid API key
  - `402` `INSUFFICIENT_CREDITS`
  - `403` missing scope, `UPGRADE_REQUIRED` (read-only plan), `WORKSPACE_SCOPE_REQUIRED`, or `ACCOUNT_ACCESS_DENIED`
  - `400` `UNKNOWN_TOOL` or a validation error (no charge)
  - `502` backend error (credits refunded)

Errors return JSON like `{ "error": "INSUFFICIENT_CREDITS", "message": "...", "upgradeUrl": "/settings/billing" }`.

## Bring your own AI

Pull normalized data with the `pull_*` tools and feed it to your own model. Synter does not lock the data in:

```typescript
const data = await runTool({
  script_name: "pull_google_ads_performance",
  platform: "google",
  customer_id: "1234567890",
  args: ["--days", "30"],
});

const analysis = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: `Analyze this ad performance:\n${JSON.stringify(data)}` }],
});
```

## See also

- [Quick Start](../../docs/quickstart.md)
- [Conversion Tracking](../../docs/guides/conversion-tracking.md)
- [Claude Plugin & MCP](../../docs/guides/claude-plugin.md)

## License

MIT
