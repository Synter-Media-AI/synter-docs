# @synterai/sdk-js

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. One SDK across **16 direct ad platforms** (Google Ads, Meta, LinkedIn, Microsoft Ads, Reddit, TikTok, X, Pinterest, Snapchat, Spotify, Amazon Ads, Amazon DSP, The Trade Desk, OpenAI Ads, DV360, StackAdapt) and **43 platforms** via the Synter backend — with full types and IDE autocomplete.

> **`0.2.0` — live on npm.** Cross-platform `campaigns.list`, `campaigns.pause`, `campaigns.updateBudget`, and `analytics.getPerformance` route natively across all 16 direct platforms (`google`, `meta`, `linkedin`, `microsoft`, `reddit`, `tiktok`, `x`, `pinterest`, `snapchat`, `spotify`, `amazon`, `amazon-dsp`, `trade-desk`, `openai`, `dv360`, `stackadapt`).

## What this talks to

Every method calls the same live, production endpoint that powers Synter's MCP server (`@synterai/mcp-server` v1.4.6) and the `synter` CLI: `POST https://synterai.com/api/v1/tools/run`. There is no separate "SDK backend" — if a method works here, it works because the exact same call already works for every MCP client (Claude, Cursor, Codex, ChatGPT) today.

## Installation

```bash
npm install @synterai/sdk-js
```

## Authentication

Get an API key at [synterai.com/developer](https://synterai.com/developer). Keys look like `syn_` followed by 32 base64url characters.

> **⚠️ Server-side only.** Your `SYNTER_API_KEY` is a secret that can spend money and modify your ad accounts. Use this SDK from a **backend** — a Node service, a Next.js API route or Server Action, an edge/serverless function. **Never** instantiate `Synter` in client-side code (React components, browser bundles, mobile apps); the key would ship to every visitor. For a React/SPA frontend, call your own backend, and have the backend call Synter.

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY!);
// or, with options:
const synterWithOpts = new Synter({
  apiKey: process.env.SYNTER_API_KEY!,
  timeout: 30_000, // ms, default 30s
  maxRetries: 3,   // default 3
});
```

## Quick Start

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY!);

// List campaigns (defaults to Google if no platform is given)
const campaigns = await synter.campaigns.list({ platform: 'google', status: 'ENABLED', limit: 10 });

// Pause or update budget across any of the 16 supported platforms
await synter.campaigns.pause({ campaign_id: 'camp_123', platform: 'meta' });
await synter.campaigns.updateBudget({ campaign_id: 'camp_123', platform: 'linkedin', daily_budget: 75 });

// Create a Google Search campaign
await synter.campaigns.createSearch({
  campaign_name: 'Q4 Launch',
  daily_budget: 50,
  keywords: ['running shoes'],
  headlines: ['Fast. Light. Yours.', 'New Season, New PR', 'Free Shipping Today'],
  descriptions: ['Premium running shoes built for speed.', 'Order today, ships free.'],
  final_url: 'https://synterai.com/get-started',
});

// Pull performance metrics ($0.00 Free Read Access)
const perf = await synter.analytics.getPerformance({ platform: 'google', date_range: 'LAST_30_DAYS' });

// Generate an AI creative
const image = await synter.creative.generateImage({
  prompt: 'a running shoe on a cloud, product photography',
});

// Escape hatch: call any of the 155+ backend scripts by name
await synter.execute('google_ads_list_audiences', { status: 'ENABLED' }, 'google');
```

## API surface

Methods are grouped by category, matching the tool catalog. Every method takes a fully-typed input object (required fields are non-optional in TypeScript) and returns `Promise<Record<string, unknown>>`.

| Namespace | Methods |
|---|---|
| `synter.campaigns` | `list`, `createSearch`, `createDisplay`, `createPmax`, `pause`, `updateBudget` |
| `synter.analytics` | `getPerformance`, `getDailySpend` |
| `synter.keywords` | `add`, `addNegative` |
| `synter.conversions` | `create`, `list`, `diagnoseTracking` |
| `synter.creative` | `generateImage`, `generateVideo` |
| `synter.meta` | `createCampaign` |
| `synter.linkedin` | `createCampaign` |
| `synter.reddit` | `createCampaign` |
| `synter.audiences` | `stageArtifact`, `sync`, `manage` |
| `synter` (top level) | `listAdAccounts`, `uploadImage`, `listLandingPages`, `execute` |

`synter.execute(scriptName, args, platform?)` is the universal escape hatch: it can call any of the 155+ backend scripts beyond the typed methods above, converting an idiomatic `{ flagName: value }` map into the CLI-flag wire format internally.

For the full method reference, see [docs.synterai.com/sdk/typescript](https://docs.synterai.com/sdk/typescript) and [docs.synterai.com/api](https://docs.synterai.com/api).

## Conversions

`synter.conversions.create()` creates a Google Ads **conversion action** (it does not track an individual conversion event):

```typescript
// Create a conversion action; returns the conversion ID + label for GTM setup
await synter.conversions.create({ name: 'Signup', category: 'SIGNUP', value: 25 });

// List existing conversion actions
await synter.conversions.list();

// Check whether tracking (gtag.js / GTM / pixel) is installed on a site
await synter.conversions.diagnoseTracking({ url: 'https://synterai.com' });
```

## Errors

```typescript
import { SynterError, SynterValidationError } from '@synterai/sdk-js';

try {
  await synter.campaigns.createSearch({ /* missing required fields */ } as any);
} catch (err) {
  if (err instanceof SynterValidationError) {
    // Caught client-side, before any network call — e.g. a missing required field.
  } else if (err instanceof SynterError) {
    // The API rejected the call: err.status, err.message, err.code, err.details
  }
}
```

The client automatically retries up to 3 times (exponential backoff, 1s base / 10s cap) on `429` responses (honoring `Retry-After`) and on network/timeout errors. Other 4xx/5xx responses are not retried.

## Cross-Platform Write Routing

- `campaigns.pause({ campaign_id, platform })` and `campaigns.updateBudget({ campaign_id, platform, daily_budget })` dispatch directly to the platform-specific script (`pause_campaign`, `meta_ads_pause_campaign`, `linkedin_ads_pause_campaign`, `microsoft_ads_pause_campaign`, `reddit_ads_pause_campaign`, `tiktok_ads_pause_campaign`, `x_ads_pause_campaign`, etc.) across all 16 direct platforms.
- For campaign creation on non-Google/Meta/LinkedIn/Reddit platforms (such as Microsoft, TikTok, X, Pinterest, Snapchat, Spotify, Amazon Ads, Amazon DSP, The Trade Desk, OpenAI Ads, DV360, StackAdapt), use `synter.execute('<platform>_create_campaign', args, platform)`.

## Links

- [Documentation](https://docs.synterai.com/sdk/typescript)
- [Quick Start](../../docs/quickstart.md)
- [Guides](../../docs/guides/README.md)

## License

MIT
