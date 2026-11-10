# Synter Documentation & SDKs

**The developer-first SDK & MCP platform for multi-platform ad management.**

Ship ads like you ship code. One unified SDK and MCP server across **43 ad platforms** (16 with direct SDK & stdio MCP routing: **Google Ads, Meta, LinkedIn, Microsoft Ads, Reddit, TikTok, X, Pinterest, Snapchat, Spotify, Amazon Ads, Amazon DSP, The Trade Desk, OpenAI Ads, DV360, StackAdapt**).

## 📚 Documentation

- [Quick Start Guide](./docs/quickstart.md) - Get started in 5 minutes
- [TypeScript SDK](./sdk/typescript/README.md) - `@synterai/sdk-js` API reference for Node.js/TypeScript (live on npm)
- [Python SDK](./sdk/python/README.md) - `synter` API reference for Python (live on PyPI)
- [Rust SDK](./sdk/rust/README.md) - `synter` crate reference (live on crates.io)
- [Go SDK](./sdk/go/README.md) - `github.com/Synter-Media-AI/synter-go` reference
- [Java SDK](./sdk/java/README.md) - `ai.syntermedia:synter-sdk` reference (live on Maven Central)
- [Guides](./docs/guides/README.md) - UTM management, conversions, agents
- [Claude Plugin](./docs/guides/claude-plugin.md) - Run Synter inside Claude Code / Claude Desktop
- [CLI](./docs/guides/cli.md) - `synter` on the command line, and the headless path for agents

Full documentation references: [docs.synterai.com/sdk](https://docs.synterai.com/sdk) and [docs.synterai.com/api](https://docs.synterai.com/api).

## 🚀 Quick Install

Every SDK talks to the same live, production endpoint that powers Synter's MCP server (`https://mcp.synterai.com/mcp` and `@synterai/mcp-server` v1.4.6) and CLI: `POST https://synterai.com/api/v1/tools/run`. Get an API key at [synterai.com/developer](https://synterai.com/developer) (keys look like `syn_` followed by 32 base64url characters).

The endpoint accepts the key under any one of three equivalent headers — `Authorization: Bearer syn_...`, `X-Synter-Key: syn_...`, or `X-API-Key: syn_...`. The SDKs send `Authorization: Bearer`; use whichever your own client makes easiest.

> **⚠️ Server-side only.** Your `SYNTER_API_KEY` is a secret that can spend money and modify your ad accounts. Use these SDKs from a backend — never ship the key in client-side code.

### TypeScript/Node.js

```bash
npm install @synterai/sdk-js
```

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY!);

// List campaigns across any of the 16 direct platforms (defaults to Google if no platform is given)
const campaigns = await synter.campaigns.list({ platform: 'google', status: 'ENABLED', limit: 10 });

// Pause or update budget across any of the 16 direct platforms (google, meta, linkedin, microsoft, reddit, tiktok, x, etc.)
await synter.campaigns.pause({ campaign_id: 'camp_123', platform: 'meta' });
await synter.campaigns.updateBudget({ campaign_id: 'camp_123', platform: 'linkedin', daily_budget: 75 });

// Create a Google Search campaign
await synter.campaigns.createSearch({
  campaign_name: 'Q4 Product Launch',
  daily_budget: 50,
  keywords: ['saas analytics', 'data platform'],
  headlines: ['Fast. Light. Yours.', 'New Season, New Data', 'Free Trial Today'],
  descriptions: ['Analytics your team will actually use.', 'Get started in minutes.'],
  final_url: 'https://synterai.com/get-started',
});

// Pull performance metrics
const perf = await synter.analytics.getPerformance({ platform: 'google', date_range: 'LAST_30_DAYS' });
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

# Pause or update budget across any of the 16 direct platforms
client.campaigns.pause(campaign_id="camp_123", platform="meta")
client.campaigns.update_budget(campaign_id="camp_123", platform="linkedin", daily_budget=75)

# Create a Google Search campaign
result = client.campaigns.create_search(
    campaign_name="Q4 Product Launch",
    daily_budget=50,
    keywords=["saas analytics", "data platform"],
    headlines=["Fast Data", "Best Analytics 2026", "Free Trial"],
    descriptions=["Analytics your team will actually use.", "Get started in minutes."],
    final_url="https://synterai.com/get-started",
)

# Pull performance metrics
perf = client.analytics.get_performance(platform="google", date_range="LAST_7_DAYS")
```

## 💰 Pricing & Multi-Tenant B2B SaaS Support

- **Free Read Access (`$0.00`)**: All reporting, campaign inspection, GA4 analytics, and dry-run (`dry_run: true`) validation calls cost **$0.00 (Free to Read)**.
- **SOLO (`$20.00/mo`)**: Includes **$20.00 in monthly platform usage value** (`$0.05` per campaign pause/budget mutation, `$0.25` per campaign creation, `$5.00` per cross-platform media plan).
- **SCALE (`$500.00/mo`)**: Includes **$500.00 in monthly platform usage value**, autonomous background schedules, and multi-workspace support.
- **CUSTOM**: Enterprise contract with dedicated multi-tenant B2B SaaS workspace provisioning, custom rate limits, and SLA.
- **Multi-Tenant B2B SaaS**: Supports both **Pattern 1 (Managed Workspace Keys `syn_ws_...` + Hosted OAuth Links)** and **Pattern 2 (Stateless BYO-Token via `X-Synter-Provider-Token` + `X-Synter-Account-Id`)** where zero customer tokens are stored in Synter.

## ✨ Features

- **43 Ad Platforms**: 16 platforms in direct SDK & stdio MCP routing (`google`, `meta`, `linkedin`, `microsoft`, `reddit`, `tiktok`, `x`, `pinterest`, `snapchat`, `spotify`, `amazon`, `amazon-dsp`, `trade-desk`, `openai`, `dv360`, `stackadapt`) + 43 total platforms via hosted HTTP MCP (`https://mcp.synterai.com/mcp`)
- **Typed methods**: Fully-typed campaign (`list`, `pause`, `updateBudget`, `createSearch`, `createDisplay`, `createPmax`), keyword, conversion, creative, and audience tools
- **Escape hatch**: `execute(scriptName, args, platform?)` reaches any of the 155+ backend scripts
- **Same surface everywhere**: TypeScript, Python, Rust, Java, and Go
- **Zero-trust safety controls**: Read-only mode (`--read-only`), focused ad-ops toolset (`--toolset ad-ops`, 32 core tools), dry-run validation (`dry_run: true`), and scoped API keys

## 🔗 Links

- [Website](https://synterai.com)
- [Documentation](https://docs.synterai.com)
- [MCP Server](https://synterai.com/mcp)
- [Developer / API keys](https://synterai.com/developer)
- [Multi-Tenant Auth Guide](https://docs.synterai.com/api/multi-tenant-auth)

## 📄 License

MIT
