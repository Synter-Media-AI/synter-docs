# synter

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. One SDK for Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, and X — backed by the same API that powers Synter's 20+ platform integrations.

## Features

✅ **Multi-platform**: Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, X
✅ **Campaign management**: Create, list, pause, and re-budget campaigns programmatically
✅ **Performance data**: Cross-platform metrics and daily spend
✅ **Creative generation**: On-brand image and video ads
✅ **Type-safe**: Full type hints for Python 3.8+
✅ **Sync and async**: `Synter` for scripts, `AsyncSynter` (httpx) for async apps

## Installation

```bash
pip install synter
```

## Quick Start

```python
from synter import Synter

client = Synter(api_key="syn_...")

# List campaigns
campaigns = client.campaigns.list(platform="google")

# Create a Google Ads search campaign
campaign = client.campaigns.create_search(
    campaign_name="Q4 Product Launch",
    daily_budget=500,
    keywords=["saas analytics", "data platform"],
    headlines=["Ship Ads Like Code", "One API for Every Platform", "Launch in Minutes"],
    descriptions=["Cross-platform ad management for developers.", "Automate campaigns across 20+ platforms."],
    final_url="https://yourapp.com",
)

# Pull performance
performance = client.analytics.get_performance(platform="google")
```

### Async

```python
from synter import AsyncSynter

async with AsyncSynter(api_key="syn_...") as client:
    campaigns = await client.campaigns.list(platform="google")
    # HTTP client automatically closed on exit
```

## API Reference

Every resource is available on both `Synter` (sync) and `AsyncSynter` (async, `await`-ed).

### `client.campaigns`

```python
client.campaigns.list(platform=..., status=..., limit=...)  # List campaigns
client.campaigns.create_search(...)                          # Google Ads search campaign
client.campaigns.create_display(...)                         # Google Ads display campaign
client.campaigns.create_pmax(...)                            # Performance Max campaign
client.campaigns.pause(campaign_id=..., platform=...)        # Pause a campaign
client.campaigns.update_budget(campaign_id=..., platform=..., daily_budget=...)  # Update budget
```

### `client.analytics`

```python
client.analytics.get_performance(platform=..., campaign_id=..., date_range=...)  # Metrics per campaign
client.analytics.get_daily_spend(days=..., platform=...)                          # Daily spend
```

### `client.conversions`

```python
client.conversions.create(name=..., value=..., category=...)  # Create a conversion action
client.conversions.list()                                      # List conversion actions
client.conversions.diagnose_tracking(url=...)                  # Check tracking on a URL
```

### `client.keywords`

```python
client.keywords.add(...)           # Add keywords to a campaign
client.keywords.add_negative(...)  # Add negative keywords
```

### `client.creative`

```python
client.creative.generate_image(...)  # Generate an image ad
client.creative.generate_video(...)  # Generate a video ad
```

### Platform-specific campaign creation

```python
client.meta.create_campaign(...)      # Meta (Facebook/Instagram)
client.linkedin.create_campaign(...)  # LinkedIn
client.reddit.create_campaign(...)    # Reddit
```

### `client.audiences`

```python
client.audiences.stage_artifact(...)  # Stage an audience artifact for upload
client.audiences.sync(...)            # Sync an audience to ad platforms
client.audiences.manage(...)          # List / attach / manage audiences
```

## Error Handling

```python
from synter import SynterError, SynterValidationError

try:
    client.campaigns.pause(campaign_id="123", platform="google")
except SynterError as e:
    print(e)  # Human-readable message
```

## Context Manager

Both clients support context managers for automatic cleanup:

```python
with Synter(api_key="syn_...") as client:
    campaigns = client.campaigns.list(platform="google")

async with AsyncSynter(api_key="syn_...") as client:
    campaigns = await client.campaigns.list(platform="google")
```

## Links

- [Documentation](https://docs.syntermedia.ai)
- [Quick Start](../../docs/quickstart.md)
- [Guides](../../docs/guides/README.md)
- [PyPI package](https://pypi.org/project/synter/)

## License

MIT
