# Quick Start Guide

Get started with Synter in 5 minutes.

## Prerequisites

- Node.js 18+ or Python 3.9+
- A Synter API key ([get one here](https://syntermedia.ai/developer))

## Installation

### TypeScript/Node.js

```bash
npm install @synterai/sdk-js
```

### Python

```bash
pip install synter
```

Other languages: [Rust](../sdk/rust/README.md) is live (`cargo add synter`); [Go](../sdk/go/README.md) and [Java](../sdk/java/README.md) are coming soon.

## Get Your API Key

1. Sign up at [syntermedia.ai](https://syntermedia.ai)
2. Go to [syntermedia.ai/developer](https://syntermedia.ai/developer)
3. Create a new API key
4. Copy the key (starts with `syn_`, followed by 32 base64url characters)

> **⚠️ Server-side only.** Your `SYNTER_API_KEY` is a secret that can spend money and modify your ad accounts. Use the SDK from a backend, never ship the key in client-side code.

## Your First Campaign

Create a Google Ads Search campaign with an ad group, responsive search ad, and keywords — in one atomic request. It requires at least 3 headlines (max 30 chars) and 2 descriptions (max 90 chars).

### TypeScript

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY!);

const result = await synter.campaigns.createSearch({
  campaign_name: 'Q4 Product Launch',
  daily_budget: 50,
  keywords: ['saas analytics', 'data platform'],
  headlines: ['Fast. Light. Yours.', 'New Season, New Data', 'Free Trial Today'],
  descriptions: ['Analytics your team will actually use.', 'Get started in minutes.'],
  final_url: 'https://example.com',
});

console.log(result);
```

### Python

```python
from synter import Synter

client = Synter(api_key="syn_...")

result = client.campaigns.create_search(
    campaign_name="Q4 Product Launch",
    daily_budget=50,
    keywords=["saas analytics", "data platform"],
    headlines=["Fast Data", "Best Analytics 2026", "Free Trial"],
    descriptions=["Analytics your team will actually use.", "Get started in minutes."],
    final_url="https://example.com",
)
print(result)
```

## List Campaigns

If no platform is given, this defaults to Google.

```typescript
const campaigns = await synter.campaigns.list({ status: 'ENABLED', limit: 10 });
```

```python
campaigns = client.campaigns.list(platform="google", status="ENABLED", limit=10)
```

## Pull Performance Metrics

Get impressions, clicks, spend, conversions, and ROAS for your campaigns.

```typescript
const perf = await synter.analytics.getPerformance({ date_range: 'LAST_30_DAYS' });
```

```python
perf = client.analytics.get_performance(platform="google", date_range="LAST_30_DAYS")
```

Valid `date_range` values: `TODAY`, `YESTERDAY`, `LAST_7_DAYS`, `LAST_30_DAYS`, `THIS_MONTH`, `LAST_MONTH`.

## Set Up Conversion Tracking

Create a Google Ads conversion action (returns the conversion ID and label for GTM setup), and check whether tracking is installed on your site.

```typescript
// Create a conversion action
await synter.conversions.create({ name: 'Signup', category: 'SIGNUP', value: 25 });

// Verify gtag.js / GTM / pixel installation
await synter.conversions.diagnoseTracking({ url: 'https://example.com' });
```

```python
client.conversions.create(name="Signup", category="SIGNUP", value=25)
client.conversions.diagnose_tracking(url="https://example.com")
```

## Generate AI Creatives

```typescript
const image = await synter.creative.generateImage({
  prompt: 'a running shoe on a cloud, product photography',
});
```

```python
image = client.creative.generate_image(prompt="a running shoe on a cloud, product photography")
```

## The Escape Hatch

The typed methods cover the 25 most common tools. To reach any of the 140+ backend scripts, use `execute`:

```typescript
await synter.execute('google_ads_list_audiences', { status: 'ENABLED' }, 'google');
```

```python
client.execute("google_ads_list_audiences", {"status": "ENABLED"}, "google")
```

## Next Steps

- [TypeScript SDK Reference](../sdk/typescript/README.md)
- [Python SDK Reference](../sdk/python/README.md)
- [Rust SDK Reference](../sdk/rust/README.md)
- [UTM Management Guide](./guides/utm-management.md)
- [Conversion Tracking Guide](./guides/conversion-tracking.md)
- [AI Agents Guide](./guides/ai-agents.md)
- Full references: [syntermedia.ai/docs/sdks](https://syntermedia.ai/docs/sdks) and [syntermedia.ai/docs/api](https://syntermedia.ai/docs/api)
