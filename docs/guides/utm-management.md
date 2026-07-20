# UTM Management

Stop hand-writing platform-specific UTM macros. The TypeScript SDK ships helper functions that translate one universal template into the right macro syntax per platform.

> **JavaScript/TypeScript only.** These UTM utilities are exported from `@synterai/sdk-js`. There is no Python equivalent today — if you need them from Python, build the query string yourself or call your Node service.

## The problem

Each ad platform has its own dynamic-parameter syntax for values like ad ID and keyword. Writing them by hand, per platform, is error-prone.

## The solution

Write one template with `{{double_brace}}` variables and let `buildUTMParameters` translate it for the target platform. The helpers currently support three platforms: **Google**, **Reddit**, and **X**.

```typescript
import { buildUTMParameters, buildTrackingURL } from '@synterai/sdk-js';

const utm = buildUTMParameters(
  {
    source: 'google',
    medium: 'cpc',
    campaign: 'product-launch',
    content: '{{ad_id}}',
    term: '{{keyword}}',
  },
  'google'
);
// → { source: 'google', medium: 'cpc', campaign: 'product-launch',
//     content: '{creative}', term: '{keyword}' }

const url = buildTrackingURL('https://example.com/landing', utm);
// → https://example.com/landing?utm_source=google&utm_medium=cpc
//   &utm_campaign=product-launch&utm_content={creative}&utm_term={keyword}
```

`buildUTMParameters` falls back to sensible defaults when a field is omitted: `source` defaults to the platform name, `medium` to `cpc`, and `campaign` to an empty string.

## Universal template variables

Only the mappings below are implemented. Variables not listed for a platform are passed through unchanged.

| Variable | Google | Reddit | X |
|----------|--------|--------|---|
| `{{campaign_id}}` | `{campaignid}` | `{{campaign_id}}` | `{campaign_id}` |
| `{{adgroup_id}}` | `{adgroupid}` | `{{adgroup_id}}` | — |
| `{{ad_id}}` | `{creative}` | `{{ad_id}}` | — |
| `{{keyword}}` | `{keyword}` | — | — |
| `{{match_type}}` | `{matchtype}` | — | — |
| `{{network}}` | `{network}` | — | — |
| `{{device}}` | `{device}` | — | — |
| `{{placement}}` | `{placement}` | — | — |
| `{{line_item_id}}` | — | — | `{line_item_id}` |
| `{{creative_id}}` | — | — | `{creative_id}` |

## Other helpers

All exported from `@synterai/sdk-js`:

- `extractUTMParameters(url)` — pull `utm_*` values back out of a URL into a partial `UTMParameters` object.
- `validateUTMTemplate(template)` — returns `{ valid, errors }`; requires `source`, `medium`, and `campaign`, and rejects values with characters outside `a-zA-Z0-9-_.{}`.
- `generateDefaultUTMTemplate(platform, campaignName)` — builds a starter template (slugifies the campaign name; adds `{{keyword}}` as the term only for `google`).

```typescript
import {
  extractUTMParameters,
  validateUTMTemplate,
  generateDefaultUTMTemplate,
} from '@synterai/sdk-js';

const template = generateDefaultUTMTemplate('google', 'Q4 Product Launch');
const { valid, errors } = validateUTMTemplate(template);
const parsed = extractUTMParameters('https://example.com/?utm_source=google&utm_medium=cpc');
```

## Best Practices

1. **Use consistent campaign names** across platforms for easier cross-platform analysis.
2. **Include `{{ad_id}}` and `{{campaign_id}}`** in your template for granular attribution.
3. **Use `term` for keywords** on Google search.
4. **Use `content` for creative variants** to A/B test ad copy.
5. **Validate before you ship** with `validateUTMTemplate`.

## Reference

- [TypeScript SDK](../../sdk/typescript/README.md)
- [Conversion Tracking](./conversion-tracking.md)
- API reference: [syntermedia.ai/docs/api](https://syntermedia.ai/docs/api)
