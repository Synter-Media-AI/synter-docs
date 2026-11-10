# synter (Python)

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. Manage **16 direct ad platforms** (Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, X, Pinterest, Snapchat, Spotify, Amazon Ads, Amazon DSP, The Trade Desk, OpenAI Ads, DV360, StackAdapt) and **43 total platforms** via `execute()` from one client.

> **`0.1.3` — live on PyPI.** `pip install synter`. Cross-platform `campaigns.list`, `campaigns.pause`, `campaigns.update_budget`, and `analytics.get_performance` route natively across all 16 direct platforms.

It talks directly to `https://synterai.com/api/v1/tools/run` — the same production endpoint the published `@synterai/mcp-server` npm package uses internally, so every call here has already been exercised in production by every MCP client (Claude, Cursor, Codex, ChatGPT).

## Status

- **Sync client (`Synter`)**: fully implemented, all 25 typed tools + `execute()` for all 155+ scripts.
- **Async client (`AsyncSynter`)**: fully implemented, mirrors the sync client's entire surface.

## Installation

```bash
pip install synter
```

Requires Python 3.9+.

> **⚠️ Server-side only.** The key is a secret that can spend money and modify ad accounts. Use this SDK from a backend, never ship it in client-side code.

## Getting an API key

Create a key at [synterai.com/developer](https://synterai.com/developer). Keys look like `syn_` followed by 32 base64url characters. Treat it like a password — anyone with it can act on your connected ad accounts.

## Quick Start

```python
from synter import Synter

client = Synter(api_key="syn_...")  # or read from an env var yourself

# List campaigns (defaults to Google if no platform is given)
campaigns = client.campaigns.list(platform="google", status="ENABLED", limit=20)

# Pause or update budget across any of the 16 direct platforms
client.campaigns.pause(campaign_id="camp_123", platform="meta")
client.campaigns.update_budget(campaign_id="camp_123", platform="linkedin", daily_budget=75)

# Create a Search campaign
result = client.campaigns.create_search(
    campaign_name="Q4 Launch",
    daily_budget=50,
    keywords=["running shoes", "trail shoes"],
    headlines=["Buy Shoes Now", "Best Shoes 2026", "Free Shipping"],
    descriptions=["Great shoes for less.", "Shop today."],
    final_url="https://synterai.com/get-started",
)

# Pull performance metrics ($0.00 Free Read Access)
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

Every backend script (155+ across all supported platforms) is reachable even without a typed method, via `execute()` — the SDK-level mirror of the `run_tool` MCP tool:

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
client.conversions.diagnose_tracking(url="https://synterai.com")
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

For the full method reference, see [docs.synterai.com/sdk/python](https://docs.synterai.com/sdk/python) and [docs.synterai.com/api](https://docs.synterai.com/api).

## Cross-Platform Write Routing

- `campaigns.pause(campaign_id=..., platform=...)` and `campaigns.update_budget(campaign_id=..., platform=..., daily_budget=...)` route natively to the platform-specific script (`pause_campaign`, `meta_ads_pause_campaign`, `linkedin_ads_pause_campaign`, `microsoft_ads_pause_campaign`, `reddit_ads_pause_campaign`, `tiktok_ads_pause_campaign`, `x_ads_pause_campaign`, etc.) across all 16 direct platforms.
- For campaign creation on non-Google/Meta/LinkedIn/Reddit platforms, use `client.execute("<platform>_create_campaign", args, platform)`.

## Links

- [Documentation](https://docs.synterai.com/sdk/python)
- [Quick Start](../../docs/quickstart.md)
- [Guides](../../docs/guides/README.md)

## License

MIT
