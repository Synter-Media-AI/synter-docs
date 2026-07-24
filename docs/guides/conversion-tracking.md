# Conversion Tracking

Set up conversion tracking across your ad platforms, with client-side utilities for click ID detection.

## Overview

Conversion tracking in Synter has two parts:

1. **Conversion actions** — defined once per event type (signup, purchase, ...) via the API, so ad platforms know what to optimize toward.
2. **Click ID capture** — client-side utilities that detect which platform a visitor came from, so conversions attribute correctly.

## Define Conversion Actions

### TypeScript

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY);

await synter.conversions.create({
  name: 'Purchase',
  value: 299.99,          // Default conversion value in USD (optional)
  category: 'PURCHASE'    // PURCHASE | SIGNUP | LEAD | PAGE_VIEW | ADD_TO_CART | DOWNLOAD | OTHER
});

// List existing conversion actions
const actions = await synter.conversions.list();
```

### Python

```python
from synter import Synter

client = Synter(api_key="syn_...")

client.conversions.create(name="Purchase", value=299.99, category="PURCHASE")
```

## Diagnose Tracking on Your Site

Check that pixels and tags are firing correctly:

```typescript
const report = await synter.conversions.diagnoseTracking({
  url: 'https://yourapp.com'
});
```

## Click ID Detection

Each platform appends a different URL parameter when a user clicks an ad:

| Platform | Parameter | Example |
|----------|-----------|---------|
| Google Ads | `gclid` | `?gclid=CjwKCAjw...` |
| Reddit | `rdt_cid` | `?rdt_cid=abc123` |
| LinkedIn | `li_fat_id` | `?li_fat_id=xyz789` |
| Microsoft | `msclkid` | `?msclkid=def456` |
| Meta | `fbclid` | `?fbclid=fb123` |
| X/Twitter | `twclid` | `?twclid=tw789` |

The SDK ships client-side utilities (no network calls) to capture these:

```typescript
import { detectClickId, extractTrackingParams, validateClickId } from '@synterai/sdk-js';

// Detect click ID from the current URL
const { platform, clickId } = detectClickId(window.location.href);
console.log(platform); // 'google'
console.log(clickId);  // 'CjwKCAjw...'

// Extract all tracking params (click IDs + UTMs)
const tracking = extractTrackingParams(window.location.href, document.referrer);
console.log(tracking.clickIds); // { google: 'abc123' }
console.log(tracking.utm);      // { source: 'google', medium: 'cpc', ... }
```

## Store Click IDs Server-Side

For accurate attribution, persist the click ID when the visitor lands and associate it with the eventual conversion:

```typescript
// On page load, store click ID in session
app.get('/', (req, res) => {
  const { platform, clickId } = detectClickId(req.originalUrl);
  if (clickId) {
    req.session.clickId = clickId;
    req.session.clickPlatform = platform;
  }
});
```

## Best Practices

1. **Define conversion actions first** — platforms need them before they can attribute or optimize
2. **Store click IDs server-side** for accurate attribution across sessions
3. **Use consistent event names** across your codebase
4. **Run `diagnoseTracking`** after any landing page change
5. **Google Analytics 4 reads are free** — use the GA4 tools (no credits) to cross-check conversion counts
