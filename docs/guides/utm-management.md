# UTM Management

Stop manually managing UTM parameters. Synter's SDK utilities translate a universal template syntax into each platform's dynamic macros.

## The Problem

Each ad platform has its own dynamic parameter syntax — Google uses `{keyword}` and `{creative}`, X uses `{campaign_id}`, Reddit uses `{{ad_id}}`. Managing these manually is error-prone and tedious.

## The Solution

Write one universal template and let the SDK translate it per platform:

```typescript
import { buildUTMParameters, buildTrackingURL } from '@synterai/sdk-js';

const utm = buildUTMParameters(
  {
    source: 'google',
    medium: 'cpc',
    campaign: 'product-launch',
    content: '{{ad_id}}',     // → {creative}
    term: '{{keyword}}'       // → {keyword}
  },
  'google'
);

const url = buildTrackingURL('https://yourapp.com/landing', utm);
// https://yourapp.com/landing?utm_source=google&utm_medium=cpc
//   &utm_campaign=product-launch&utm_content={creative}&utm_term={keyword}
```

## Universal Template Variables

| Variable | Google | Reddit | X |
|----------|--------|--------|---|
| `{{campaign_id}}` | `{campaignid}` | `{{campaign_id}}` | `{campaign_id}` |
| `{{adgroup_id}}` | `{adgroupid}` | `{{adgroup_id}}` | — |
| `{{ad_id}}` | `{creative}` | `{{ad_id}}` | — |
| `{{keyword}}` | `{keyword}` | — | — |
| `{{device}}` | `{device}` | — | — |
| `{{placement}}` | `{placement}` | — | — |
| `{{creative_id}}` | — | — | `{creative_id}` |

Unset `source`/`medium` fall back to sensible defaults (`source` = the platform, `medium` = `cpc`).

## More Utilities

```typescript
import {
  extractUTMParameters,     // Parse UTM params out of a URL
  validateUTMTemplate,      // Validate a template before use
  generateDefaultUTMTemplate // Sensible defaults for a platform + campaign
} from '@synterai/sdk-js';
```

All UTM utilities run client-side — no network calls, no credits.

## Best Practices

1. **Use consistent campaign names** across platforms for easier cross-platform analysis
2. **Include ad_id and campaign_id** in UTMs for granular attribution
3. **Use term for keywords** on search platforms
4. **Use content for creative variants** to A/B test different ad copy
