# @synterai/sdk-js

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. One SDK for Google Ads, Meta, LinkedIn, Microsoft Ads, Reddit, TikTok, and X — with full types and IDE autocomplete.

> **`0.1.2` — live on npm.** Pre-1.0: the surface may change before a stable `1.0`. Pin a version in production. `0.1.2` fixes `analytics.getPerformance({ platform: "google" | "linkedin" })`, which in `0.1.0`/`0.1.1` sent the script filename (`pull_google_ads_data`) instead of the canonical script name (`pull_google_ads`).

## What this talks to

Every method calls the same live, production endpoint that powers Synter's MCP server (`@synterai/mcp-server`) and the `synter` CLI: `POST https://synterai.com/api/v1/tools/run`. There is no separate "SDK backend" — if a method works here, it works because the exact same call already works for every MCP client (Claude, Cursor, Codex, ChatGPT) today.

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
const campaigns = await synter.campaigns.list({ status: 'ENABLED', limit: 10 });

// Create a Google Search campaign
await synter.campaigns.createSearch({
  campaign_name: 'Q4 Launch',
  daily_budget: 50,
  keywords: ['running shoes'],
  headlines: ['Fast. Light. Yours.', 'New Season, New PR', 'Free Shipping Today'],
  descriptions: ['Premium running shoes built for speed.', 'Order today, ships free.'],
  final_url: 'https://example.com/shoes',
});

// Pull performance metrics
const perf = await synter.analytics.getPerformance({ date_range: 'LAST_30_DAYS' });

// Generate an AI creative
const image = await synter.creative.generateImage({
  prompt: 'a running shoe on a cloud, product photography',
});

// Escape hatch: call any of the 140+ backend scripts by name
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

`synter.execute(scriptName, args, platform?)` is the universal escape hatch: it can call any of the 140+ backend scripts beyond the typed methods above, converting an idiomatic `{ flagName: value }` map into the CLI-flag wire format internally.

For the full method reference, see [docs.synterai.com/sdk](https://docs.synterai.com/sdk) and [docs.synterai.com/api](https://docs.synterai.com/api/authentication).

## Conversions

`synter.conversions.create()` creates a Google Ads **conversion action** (it does not track an individual conversion event):

```typescript
// Create a conversion action; returns the conversion ID + label for GTM setup
await synter.conversions.create({ name: 'Signup', category: 'SIGNUP', value: 25 });

// List existing conversion actions
await synter.conversions.list();

// Check whether tracking (gtag.js / GTM / pixel) is installed on a site
await synter.conversions.diagnoseTracking({ url: 'https://example.com' });
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

## Known issues (intentionally preserved, not "fixed" here)

These mirror real, current production behavior of the backend scripts. The SDK calls the backend exactly as it works today rather than silently diverging:

1. `campaigns.pause()` and `campaigns.updateBudget()` accept a `platform` field (their schemas advertise all 7 platforms), but the live dispatch always uses `platform=google` regardless of what's passed.
2. `uploadImage()` accepts an optional `platform` (google/meta/linkedin), but live dispatch always hardcodes `platform=google`.
3. `campaigns.createDisplay()` and `campaigns.createPmax()` use different flag names for the same concept — `--landscape-image`/`--square-image` vs. `--landscape-image-url`/`--square-image-url`.

## Links

- [Documentation](https://docs.synterai.com/sdk)
- [Quick Start](../../docs/quickstart.md)
- [Guides](../../docs/guides/README.md)

## Write paths and approvals

Copy-paste examples for pausing a campaign and changing a daily budget, and how a held write comes back (HTTP 202 `pending_review` with an `audit_id`), are on [docs.synterai.com/sdk/typescript](https://docs.synterai.com/sdk/typescript). Which writes run immediately and which wait for a person on each surface: [Approval model](https://docs.synterai.com/mcp/approval-model).

**Known issue (0.1.x):** `updateBudget`/`update_budget` sends `--budget`, which the API rejects with HTTP 400. The fix is in the SDK source and ships in the next release; until then use the generic `execute` call shown in the docs page above.

## License

MIT
