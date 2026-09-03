# synter (Python)

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. Manage Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, and X ad campaigns from one client.

> **`0.1.2` — live on PyPI.** `pip install synter`. Pre-1.0, so the surface may change before a stable `1.0`; pin a version in production. `0.1.1`+ fixes `analytics.get_performance(platform="google" | "linkedin")`, which in `0.1.0` sent the script filename (`pull_google_ads_data`) instead of the canonical script name (`pull_google_ads`).

It talks directly to `https://syntermedia.ai/api/v1/tools/run` — the same production endpoint the published `@synterai/mcp-server` npm package uses internally, so every call here has already been exercised in production by every MCP client (Claude, Cursor, Codex, ChatGPT).

## Status

- **Sync client (`Synter`)**: fully implemented, all 25 tools.
- **Async client (`AsyncSynter`)**: fully implemented, mirrors the sync client's entire surface.

## Installation

```bash
pip install synter
```

Requires Python 3.9+.

> **⚠️ Server-side only.** The key is a secret that can spend money and modify ad accounts. Use this SDK from a backend, never ship it in client-side code.

## Getting an API key

Create a key at [syntermedia.ai/developer](https://syntermedia.ai/developer). Keys look like `syn_` followed by 32 base64url characters. Treat it like a password — anyone with it can act on your connected ad accounts.

## Quick Start

```python
from synter import Synter

client = Synter(api_key="syn_...")  # or read from an env var yourself

# List campaigns (defaults to Google if no platform is given)
campaigns = client.campaigns.list(platform="google", status="ENABLED", limit=20)

# Create a Search campaign
result = client.campaigns.create_search(
    campaign_name="Q4 Launch",
    daily_budget=50,
    keywords=["running shoes", "trail shoes"],
    headlines=["Buy Shoes Now", "Best Shoes 2026", "Free Shipping"],
    descriptions=["Great shoes for less.", "Shop today."],
    final_url="https://example.com/shoes",
)

# Pull performance metrics
perf = client.analytics.get_performance(platform="google", date_range="LAST_7_DAYS")
```

### Async usage

```python
import asyncio
from synter import AsyncSynter

async def main():
    async with AsyncSynter(api_key="syn_...") as client:
        campaigns = await client.campaigns.list(platform="google")
        print(campaigns)

asyncio.run(main())
```

### The escape hatch: `execute()`

Every backend script (140+ across 19 platforms) is reachable even without a typed method, via `execute()` — the SDK-level mirror of the `run_tool` MCP tool:

```python
# Idiomatic dict form (recommended): converted to CLI flags automatically.
client.execute("google_ads_list_audiences", {"account_id": "123", "status": "ENABLED"})

# Raw CLI-flag-array form, for full parity with the underlying backend contract.
client.execute("google_ads_list_audiences", ["--account-id", "123", "--status", "ENABLED"])
```

## Conversions

`client.conversions.create()` creates a Google Ads **conversion action** (it does not track an individual conversion event):

```python
# Create a conversion action; returns the conversion ID + label for GTM setup
client.conversions.create(name="Signup", category="SIGNUP", value=25)

# List existing conversion actions
client.conversions.list()

# Check whether tracking (gtag.js / GTM / pixel) is installed on a site
client.conversions.diagnose_tracking(url="https://example.com")
```

## Error handling

Every failure raises `synter.SynterError` (or its subclass `synter.SynterValidationError` for client-side validation failures caught before any network call):

```python
from synter import Synter, SynterError, SynterValidationError

client = Synter(api_key="syn_...")

try:
    client.campaigns.create_search(campaign_name="", daily_budget=50)
except SynterValidationError as e:
    print("You called it wrong:", e.message)
except SynterError as e:
    print(f"API error [{e.status}] {e.message} (code={e.code})")
```

`SynterError` carries `message`, `status` (HTTP status, or `0` for network/timeout failures), `code`, and `details`.

## Retries

Requests are retried up to 3 times with exponential backoff (1s base, 10s cap), only for HTTP 429 (honoring the `Retry-After` header) and network/timeout errors. Other 4xx responses are never retried. Default request timeout is 30 seconds; override with `Synter(api_key=..., timeout=60.0)`.

## Resource namespaces

| Namespace | Methods |
|---|---|
| `client.campaigns` | `list`, `create_search`, `create_display`, `create_pmax`, `pause`, `update_budget` |
| `client.analytics` | `get_performance`, `get_daily_spend` |
| `client.keywords` | `add`, `add_negative` |
| `client.conversions` | `create`, `list`, `diagnose_tracking` |
| `client.creative` | `generate_image`, `generate_video` |
| `client.meta` | `create_campaign` |
| `client.linkedin` | `create_campaign` |
| `client.reddit` | `create_campaign` |
| `client.audiences` | `stage_artifact`, `sync`, `manage` |
| `client.*` (top level) | `list_ad_accounts`, `upload_image`, `list_landing_pages`, `execute` |

For the full method reference, see [syntermedia.ai/docs/sdks](https://syntermedia.ai/docs/sdks) and [syntermedia.ai/docs/api](https://syntermedia.ai/docs/api).

## Known issues (preserved, not silently fixed)

This SDK calls the real backend exactly as it behaves in production today, including three known quirks:

1. `campaigns.pause()` and `campaigns.update_budget()` accept a `platform` argument (schema parity) but always dispatch with `platform=google` regardless of what you pass.
2. `upload_image()` accepts an optional `platform` argument but always dispatches with `platform=google`.
3. `campaigns.create_display()` uses backend flags `--landscape-image`/`--square-image`, while `campaigns.create_pmax()` uses `--landscape-image-url`/`--square-image-url`. The wire flags genuinely differ because the two backend scripts differ.

## Links

- [Documentation](https://syntermedia.ai/docs/sdks)
- [Quick Start](../../docs/quickstart.md)
- [Guides](../../docs/guides/README.md)

## License

MIT
