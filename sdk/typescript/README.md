# @synterai/sdk-js

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. One SDK for Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, and X — backed by the same API that powers Synter's 20+ platform integrations.

## Features

✅ **Multi-platform**: Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, X
✅ **Campaign management**: Create, list, pause, and re-budget campaigns programmatically
✅ **Performance data**: Cross-platform metrics and daily spend
✅ **Creative generation**: On-brand image and video ads
✅ **UTM & conversion utilities**: Client-side helpers for tracking, free of network calls
✅ **TypeScript-native**: Full type safety and IntelliSense

## Installation

```bash
npm install @synterai/sdk-js
```

## Quick Start

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY);

// List campaigns
const campaigns = await synter.campaigns.list({ platform: 'google' });

// Create a Google Ads search campaign
const campaign = await synter.campaigns.createSearch({
  campaign_name: 'Q4 Product Launch',
  daily_budget: 500,
  keywords: ['saas analytics', 'data platform'],
  headlines: ['Ship Ads Like Code', 'One API for Every Platform', 'Launch in Minutes'],
  descriptions: ['Cross-platform ad management for developers.', 'Automate campaigns across 20+ platforms.'],
  final_url: 'https://yourapp.com',
});

// Pull performance
const performance = await synter.analytics.getPerformance({
  platform: 'google',
  date_range: 'LAST_7_DAYS'
});
```

## API Reference

### `synter.campaigns`

```typescript
await synter.campaigns.list(input)          // List campaigns (platform, status, limit)
await synter.campaigns.createSearch(input)  // Create a Google Ads search campaign
await synter.campaigns.createDisplay(input) // Create a Google Ads display campaign
await synter.campaigns.createPmax(input)    // Create a Performance Max campaign
await synter.campaigns.pause(input)         // Pause a campaign ({ campaign_id, platform })
await synter.campaigns.updateBudget(input)  // Update daily budget ({ campaign_id, platform, daily_budget })
```

### `synter.analytics`

```typescript
await synter.analytics.getPerformance(input) // Metrics per campaign ({ platform?, campaign_id?, date_range? })
await synter.analytics.getDailySpend(input)  // Daily spend ({ days?, platform? })
```

### `synter.conversions`

```typescript
await synter.conversions.create(input)           // Create a conversion action ({ name, value?, category? })
await synter.conversions.list()                  // List conversion actions
await synter.conversions.diagnoseTracking(input) // Check tracking on a URL ({ url })
```

### `synter.keywords`

```typescript
await synter.keywords.add(input)         // Add keywords to a campaign
await synter.keywords.addNegative(input) // Add negative keywords
```

### `synter.creative`

```typescript
await synter.creative.generateImage(input) // Generate an image ad
await synter.creative.generateVideo(input) // Generate a video ad
```

### Platform-specific campaign creation

```typescript
await synter.meta.createCampaign(input)     // Meta (Facebook/Instagram)
await synter.linkedin.createCampaign(input) // LinkedIn
await synter.reddit.createCampaign(input)   // Reddit
```

### `synter.audiences`

```typescript
await synter.audiences.stageArtifact(input) // Stage an audience artifact for upload
await synter.audiences.sync(input)          // Sync an audience to ad platforms
await synter.audiences.manage(input)        // List / attach / manage audiences
```

## UTM Utilities

Client-side helpers — no network calls, no credits:

```typescript
import { buildUTMParameters, buildTrackingURL, validateUTMTemplate } from '@synterai/sdk-js';

const utm = buildUTMParameters(
  {
    source: 'google',
    campaign: 'launch',
    content: '{{ad_id}}_{{keyword}}'
  },
  'google'
);
// → { content: '{creative}_{keyword}' }
```

## Conversion Tracking Utilities

```typescript
import { detectClickId, extractTrackingParams } from '@synterai/sdk-js';

// Detect click ID from URL (gclid, rdt_cid, twclid, ...)
const { platform, clickId } = detectClickId(window.location.href);

// Extract all tracking params
const tracking = extractTrackingParams(window.location.href, document.referrer);
console.log(tracking.clickIds); // { google: 'abc123', ... }
console.log(tracking.utm);      // { source: 'google', medium: 'cpc', ... }
```

## Error Handling

```typescript
import { SynterError, SynterValidationError } from '@synterai/sdk-js';

try {
  await synter.campaigns.pause({ campaign_id: '123', platform: 'google' });
} catch (error) {
  if (error instanceof SynterError) {
    console.error(error.message); // Human-readable message
  }
}
```

## TypeScript Support

Full type safety out of the box:

```typescript
import type { CreateSearchCampaignInput, AdPlatform, DateRange } from '@synterai/sdk-js';
```

## Links

- [Documentation](https://docs.syntermedia.ai)
- [Quick Start](../../docs/quickstart.md)
- [Guides](../../docs/guides/README.md)
- [npm package](https://www.npmjs.com/package/@synterai/sdk-js)

## License

MIT
