# Synter REST API from Python

There is no published `synter` PyPI package. Synter's programmatic surface is one authenticated REST endpoint that runs any Synter tool. You call it with `requests` or `httpx`. For agent-driven use in Claude, Cursor, or Codex, use the [MCP server](../../docs/guides/claude-plugin.md) instead.

## Setup

- **Base URL:** `https://syntermedia.ai/api/v1`
- **Auth:** `Authorization: Bearer syn_...` (create a key at [syntermedia.ai/developer](https://syntermedia.ai/developer))

A tiny wrapper is all you need:

```python
import os
import requests

BASE = "https://syntermedia.ai/api/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['SYNTER_API_KEY']}"}


def run_tool(script_name, args=None, platform=None, customer_id=None):
    body = {"script_name": script_name}
    if args:
        body["args"] = args
    if platform:
        body["platform"] = platform
    if customer_id:
        body["customer_id"] = customer_id
    res = requests.post(f"{BASE}/tools/run", headers=HEADERS, json=body, timeout=120)
    data = res.json()
    if not res.ok:
        raise RuntimeError(f"Synter {res.status_code}: {data}")
    return data
```

## List available tools

```python
res = requests.get(f"{BASE}/tools/run", headers=HEADERS, timeout=30)
print(res.json()["tools"])
```

Requires the `tools:read` scope. The list reflects the tools your key's plan and workspace can run.

## Run a tool

`POST /api/v1/tools/run`:

| Field | Required | Description |
|-------|----------|-------------|
| `script_name` | yes | Tool name (lowercase, underscores only). |
| `args` | no | List of string CLI-style arguments. |
| `platform` | no | `google`, `meta`, `linkedin`, `reddit`, `microsoft`, `tiktok`, `x`, ... |
| `customer_id` | no | Ad account ID; must be connected in this key's workspace. |

```python
# Pull recent performance (read; needs tools:read)
perf = run_tool(
    "pull_google_ads_performance",
    platform="google",
    customer_id="1234567890",
    args=["--days", "30"],
)

# Upload offline conversions from a CSV (write; needs tools:write, deducts credits)
upload = run_tool(
    "google_ads_upload_offline_conversions",
    platform="google",
    customer_id="1234567890",
    args=[
        "--csv-url", "https://yourapp.com/exports/conversions.csv",
        "--conversion-name", "Offline Purchase",
        # add "--dry-run" first to validate without uploading
    ],
)
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

## Bring your own AI

Pull normalized data with the `pull_*` tools and feed it to your own model:

```python
data = run_tool(
    "pull_google_ads_performance",
    platform="google",
    customer_id="1234567890",
    args=["--days", "30"],
)
# hand `data` to your model, warehouse, or notebook
```

## See also

- [Quick Start](../../docs/quickstart.md)
- [Conversion Tracking](../../docs/guides/conversion-tracking.md)
- [Claude Plugin & MCP](../../docs/guides/claude-plugin.md)

## License

MIT
