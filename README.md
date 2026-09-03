# Synter Documentation

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. One SDK for Google Ads, Reddit Ads, LinkedIn Ads, Microsoft Ads, Meta Ads, TikTok Ads, and X Ads.

## 📚 Documentation

- [Quick Start Guide](./docs/quickstart.md) - Get started in 5 minutes
- [TypeScript SDK](./sdk/typescript/README.md) - `@synterai/sdk-js` API reference for Node.js/TypeScript (live on npm)
- [Python SDK](./sdk/python/README.md) - `synter` API reference for Python (live on PyPI)
- [Rust SDK](./sdk/rust/README.md) - `synter` crate reference (live on crates.io)
- [Go SDK](./sdk/go/README.md) - coming soon
- [Java SDK](./sdk/java/README.md) - `ai.syntermedia:synter-sdk` reference (live on Maven Central)
- [Guides](./docs/guides/README.md) - UTM management, conversions, agents
- [Claude Plugin](./docs/guides/claude-plugin.md) - Run Synter inside Claude Code / Claude Desktop
- [CLI](./docs/guides/cli.md) - `synter` on the command line, and the headless path for agents

Full in-app references: [syntermedia.ai/docs/sdks](https://syntermedia.ai/docs/sdks) and [syntermedia.ai/docs/api](https://syntermedia.ai/docs/api).

## 🚀 Quick Install

Every SDK talks to the same live, production endpoint that powers Synter's MCP server and CLI: `POST https://syntermedia.ai/api/v1/tools/run`. Get an API key at [syntermedia.ai/developer](https://syntermedia.ai/developer) (keys look like `syn_` followed by 32 base64url characters).

The endpoint accepts the key under any one of three equivalent headers — `Authorization: Bearer syn_...`, `X-Synter-Key: syn_...`, or `X-API-Key: syn_...`. The SDKs send `Authorization: Bearer`; use whichever your own client makes easiest.

> **⚠️ Server-side only.** Your `SYNTER_API_KEY` is a secret that can spend money and modify your ad accounts. Use these SDKs from a backend — never ship the key in client-side code.

### TypeScript/Node.js

```bash
npm install @synterai/sdk-js
```

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY!);

// List campaigns (defaults to Google if no platform is given)
const campaigns = await synter.campaigns.list({ status: 'ENABLED', limit: 10 });

// Create a Google Search campaign
await synter.campaigns.createSearch({
  campaign_name: 'Q4 Product Launch',
  daily_budget: 50,
  keywords: ['saas analytics', 'data platform'],
  headlines: ['Fast. Light. Yours.', 'New Season, New Data', 'Free Trial Today'],
  descriptions: ['Analytics your team will actually use.', 'Get started in minutes.'],
  final_url: 'https://example.com',
});

// Pull performance metrics
const perf = await synter.analytics.getPerformance({ date_range: 'LAST_30_DAYS' });
```

### Python

```bash
pip install synter
```

```python
from synter import Synter

client = Synter(api_key="syn_...")

# List campaigns (defaults to Google if no platform is given)
campaigns = client.campaigns.list(platform="google", status="ENABLED", limit=20)

# Create a Google Search campaign
result = client.campaigns.create_search(
    campaign_name="Q4 Product Launch",
    daily_budget=50,
    keywords=["saas analytics", "data platform"],
    headlines=["Fast Data", "Best Analytics 2026", "Free Trial"],
    descriptions=["Analytics your team will actually use.", "Get started in minutes."],
    final_url="https://example.com",
)

# Pull performance metrics
perf = client.analytics.get_performance(platform="google", date_range="LAST_7_DAYS")
```

## ✨ Features

- **Multi-platform**: Google, Reddit, LinkedIn, Microsoft, Meta, TikTok, X
- **Typed methods**: Fully-typed campaign, keyword, conversion, creative, and audience tools
- **Escape hatch**: `execute(scriptName, args, platform?)` reaches any of the 140+ backend scripts
- **Same surface everywhere**: TypeScript, Python, Rust, and Java ship today; Go is coming soon
- **Production-tested transport**: every call already runs in production via the MCP server (Claude, Cursor, Codex, ChatGPT)

## 🔗 Links

- [Website](https://syntermedia.ai)
- [Dashboard](https://syntermedia.ai/dashboard)
- [Developer / API keys](https://syntermedia.ai/developer)
- [API Status](https://status.syntermedia.ai)
- [Discord Community](https://discord.gg/syntermedia)

## 📄 License

MIT
