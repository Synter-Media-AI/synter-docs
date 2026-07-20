# Conversion Tracking

Set up conversion actions, verify your tracking is installed, and detect ad click IDs — all against the real Synter SDK surface.

## Overview

The SDK's `conversions` namespace does two things today:

1. **Create and list Google Ads conversion actions** — `conversions.create` / `conversions.list`.
2. **Diagnose tracking installation** — `conversions.diagnoseTracking` checks whether gtag.js, GTM, and a pixel are actually present on a page.

In addition, the TypeScript SDK ships **client-side helper functions** for detecting ad click IDs and building/normalizing conversion payloads from an incoming request URL. Those helpers are JavaScript/TypeScript only — there is no Python equivalent.

> **⚠️ Server-side only.** Your `SYNTER_API_KEY` can spend money and modify ad accounts. Call the SDK from a backend, never from client-side code.

## Create a conversion action

Creates a conversion action in Google Ads and returns its ID and label for GTM setup.

### TypeScript

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY!);

await synter.conversions.create({
  name: 'Signup',        // required
  category: 'SIGNUP',    // optional, default: LEAD
  value: 25,             // optional, default conversion value in USD
});
```

### Python

```python
from synter import Synter

client = Synter(api_key="syn_...")

client.conversions.create(name="Signup", category="SIGNUP", value=25)
```

### Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | string | yes | e.g. `Signup`, `Purchase` |
| `value` | number | no | Default conversion value in USD |
| `category` | string | no | One of `PURCHASE`, `SIGNUP`, `LEAD`, `PAGE_VIEW`, `ADD_TO_CART`, `DOWNLOAD`, `OTHER` (default: `LEAD`) |

## List conversion actions

```typescript
await synter.conversions.list();
```

```python
client.conversions.list()
```

## Diagnose tracking

Check whether conversion tracking (gtag.js, GTM, pixel) is properly installed on a page.

```typescript
await synter.conversions.diagnoseTracking({ url: 'https://example.com' });
```

```python
client.conversions.diagnose_tracking(url="https://example.com")
```

## Click ID detection (TypeScript only)

When a visitor lands on your site after clicking an ad, the ad platform appends a click ID to the URL. The TypeScript SDK exports helpers to detect and extract those parameters. The helpers currently understand three platforms:

| Platform | Parameter(s) |
|----------|--------------|
| Google Ads | `gclid`, `wbraid`, `gbraid` |
| Reddit | `rdt_cid` |
| X / Twitter | `twclid` |

```typescript
import { detectClickId, extractTrackingParams } from '@synterai/sdk-js';

// Detect the single click ID present on an incoming URL
const { platform, clickId, paramName } = detectClickId(window.location.href);
console.log(platform); // 'google' | 'reddit' | 'x' | null
console.log(clickId);  // 'CjwKCAjw...' | null

// Extract all click IDs + UTM parameters from a URL (with optional referrer)
const tracking = extractTrackingParams(window.location.href, document.referrer);
console.log(tracking.clickIds); // { google: 'abc123' }
console.log(tracking.utm);      // { source: 'google', medium: 'cpc', ... }
```

Three more pure-function helpers round out the set:

- `validateClickId(platform, clickId)` — sanity-checks a click ID's format for `google`, `reddit`, or `x`.
- `buildConversionPayload({ platform, clickId, event, value?, currency?, timestamp?, orderId? })` — assembles a normalized `ConversionPayload` (defaults `currency` to `USD` and `conversionTime` to now).
- `normalizeEventName(eventName, platform)` — maps common event names (`purchase`, `signup`, `add-to-cart`, `page-view`) to each platform's naming for `google`, `reddit`, or `x`.

These helpers build and normalize data locally; they do not upload anything on their own.

## Uploading conversions to platforms

To push offline or server-side conversions into a platform's API, use the SDK's universal escape hatch, `execute(scriptName, args, platform?)`, with the appropriate backend script. Browse the available scripts at [syntermedia.ai/docs/tools](https://syntermedia.ai/docs/tools):

```typescript
// Example shape — pick the correct backend script for your platform from the tools catalog
await synter.execute('<conversion_upload_script>', { /* flag: value */ }, 'google');
```

## Best Practices

1. **Store click IDs server-side** for accurate offline attribution.
2. **Use consistent conversion action names** across your codebase.
3. **Verify installation with `diagnoseTracking`** before you rely on the data.
4. **Send conversions as soon as possible** after they occur.

## Reference

- [TypeScript SDK](../../sdk/typescript/README.md)
- [Python SDK](../../sdk/python/README.md)
- Full tool catalog: [syntermedia.ai/docs/tools](https://syntermedia.ai/docs/tools)
- API reference: [syntermedia.ai/docs/api](https://syntermedia.ai/docs/api)
