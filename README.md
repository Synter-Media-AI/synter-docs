# Synter Documentation

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. One SDK for Google Ads, Reddit Ads, LinkedIn Ads, Microsoft Ads, Meta Ads, and X Ads.

## 📚 Documentation

- [Quick Start Guide](./docs/quickstart.md) - Get started in 5 minutes
- [TypeScript SDK](./sdk/typescript/README.md) - Full API reference for Node.js/TypeScript
- [Python SDK](./sdk/python/README.md) - Full API reference for Python
- [Guides](./docs/guides/README.md) - UTM management, conversions, agents
- [Claude Plugin](./docs/guides/claude-plugin.md) - Run Synter inside Claude Code / Claude Desktop

## 🚀 Quick Install

### TypeScript/Node.js

```bash
npm install @synter/sdk
```

```typescript
import { Synter } from '@synter/sdk';

const synter = new Synter(process.env.SYNTER_API_KEY);

// Create a campaign
const campaign = await synter.campaigns.create({
  name: 'Q4 Product Launch',
  platform: 'google',
  budget: { daily: 500, currency: 'USD' }
});

// Track conversions
await synter.conversions.create({
  event: 'purchase',
  value: 299.99
});
```

### Python

```bash
pip install synter
```

```python
from synter import Synter

async with Synter(api_key="syn_...") as synter:
    # Create a campaign
    campaign = await synter.campaigns.create({
        "name": "Q4 Product Launch",
        "platform": "google",
        "budget": {"daily": 500, "currency": "USD"}
    })

    # Track conversions
    await synter.conversions.create({
        "event": "purchase",
        "value": 299.99
    })
```

## ✨ Features

- **Multi-platform**: Google, Reddit, LinkedIn, Microsoft, Meta, X
- **UTM Management**: Auto-generated, platform-specific UTM parameters
- **Conversion Tracking**: Unified tracking across all platforms
- **AI Agents**: Transparent, controllable budget optimization
- **Analytics Integration**: PostHog, Mixpanel, Heap, Segment
- **Bring Your Own AI**: Export raw data for custom ML models

## 🔗 Links

- [Website](https://syntermedia.ai)
- [Dashboard](https://syntermedia.ai/dashboard)
- [API Status](https://status.syntermedia.ai)
- [Discord Community](https://discord.gg/syntermedia)

## 📄 License

MIT
