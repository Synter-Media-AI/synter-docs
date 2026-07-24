# Quick Start Guide

Get started with Synter in 5 minutes.

## Prerequisites

- Node.js 18+ or Python 3.8+
- A Synter API key ([get one from the Developer Portal](https://syntermedia.ai/developer))

## Installation

### TypeScript/Node.js

```bash
npm install @synterai/sdk-js
```

### Python

```bash
pip install synter
```

### CLI

```bash
npm install -g synter
synter login   # opens a browser device-code flow — no copy-paste needed
```

## Get Your API Key

1. Sign up at [syntermedia.ai](https://syntermedia.ai)
2. Open the [Developer Portal](https://syntermedia.ai/developer)
3. Create a new API key
4. Copy the key (starts with `syn_`)

**No account yet?** If you're connecting from an MCP client (Claude, Cursor, etc.), you can onboard without leaving the conversation: call `synter_onboarding_start(email="you@work-email.com")`, click the magic link in your inbox, then check `synter_onboarding_status(session_token)` until your key is ready.

## Your First Campaign

### TypeScript

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY);

// Create a Google Ads search campaign
const campaign = await synter.campaigns.createSearch({
  campaign_name: 'Q4 Product Launch',
  daily_budget: 500,
  keywords: ['saas analytics', 'data platform'],
  headlines: ['Ship Ads Like Code', 'One API for Every Platform', 'Launch in Minutes'],
  descriptions: ['Cross-platform ad management for developers.', 'Automate campaigns across 20+ platforms.'],
  final_url: 'https://yourapp.com',
  geo_targets: ['US', 'CA']
});
```

### Python

```python
from synter import Synter

client = Synter(api_key="syn_...")

campaign = client.campaigns.create_search(
    campaign_name="Q4 Product Launch",
    daily_budget=500,
    keywords=["saas analytics", "data platform"],
    headlines=["Ship Ads Like Code", "One API for Every Platform", "Launch in Minutes"],
    descriptions=["Cross-platform ad management for developers.", "Automate campaigns across 20+ platforms."],
    final_url="https://yourapp.com",
    geo_targets=["US", "CA"],
)
```

Prefer async? Use `AsyncSynter`:

```python
from synter import AsyncSynter

async with AsyncSynter(api_key="syn_...") as client:
    campaigns = await client.campaigns.list(platform="google")
```

## Pull Performance

```typescript
const performance = await synter.analytics.getPerformance({
  platform: 'google',
  date_range: 'LAST_7_DAYS'
});

const spend = await synter.analytics.getDailySpend({ days: 7 });
```

## Set Up Conversion Tracking

Define a conversion action, then verify your site's tracking:

```typescript
// Create a conversion action
await synter.conversions.create({
  name: 'Purchase',
  value: 299.99,
  category: 'PURCHASE'
});

// Diagnose tracking on your site
await synter.conversions.diagnoseTracking({ url: 'https://yourapp.com' });
```

## Manage Campaigns

```typescript
// List campaigns
const campaigns = await synter.campaigns.list({ platform: 'google', status: 'ENABLED' });

// Pause a campaign
await synter.campaigns.pause({ campaign_id: '123456', platform: 'google' });

// Update budget
await synter.campaigns.updateBudget({
  campaign_id: '123456',
  platform: 'google',
  daily_budget: 750
});
```

## Connect Your AI Agent (MCP)

Point any MCP client at the hosted server:

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

Google Analytics 4 tools are free and cost no credits — a good first call to verify your setup.

## Next Steps

- [TypeScript SDK Reference](../sdk/typescript/README.md)
- [Python SDK Reference](../sdk/python/README.md)
- [UTM Management Guide](./guides/utm-management.md)
- [Conversion Tracking Guide](./guides/conversion-tracking.md)
- [AI Agents Guide](./guides/ai-agents.md)
- [Full docs at docs.syntermedia.ai](https://docs.syntermedia.ai)
