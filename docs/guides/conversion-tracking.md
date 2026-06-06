# Conversion Tracking

Unified conversion tracking across all ad platforms — server-side pixel forwarding and automatic click ID detection.

## Overview

There are two ways to get conversion data into your ad platforms with Synter:

1. **Pixel Destinations** (recommended) — Install the Synter JS pixel on your site. Synter captures every event server-side and forwards it to your connected ad platforms automatically.
2. **SDK / API** — Call `synter.conversions.create()` directly from your backend on each conversion event.

---

## Pixel Destinations (Server-Side Forwarding)

Install the Synter pixel once and route events to any number of ad platforms without touching each platform's SDK.

### How It Works

1. Add the Synter pixel to your site (one-time).
2. Go to **Settings → Tracking → Destinations** and add each ad platform.
3. Provide the required setting (pixel ID, conversion rule ID, etc.) for each platform.
4. Synter forwards every captured event to each platform's CAPI automatically.

### Required Settings Per Platform

| Platform | Required Setting | Where to Find It |
|---|---|---|
| Meta | `pixel_id` (15–16 digit number) | Meta Business Suite → Events Manager → Pixels |
| Google Ads | `pixel_id` (format: `AW-XXXXXXXXXX`) | Google Ads → Goals → Conversions → Tag setup |
| LinkedIn | `conversion_rule_id` (numeric) | Campaign Manager → Analyze → Conversion tracking → click rule → URL |
| Reddit | `pixel_id` (format: `a2_XXXXXXXXXXXX`) | Reddit Ads → Events → Pixel, or `rdt('init', ...)` on your site |
| TikTok | `pixel_code` (alphanumeric) | TikTok Ads Manager → Assets → Events → Web Events |
| Microsoft | `pixel_id` (numeric) | Microsoft Ads → Conversion tracking |
| Snapchat | `pixel_id` (UUID) | Snapchat Ads Manager → Events Manager |
| Pinterest | `pixel_id` (numeric) | Pinterest Ads → Conversions |
| X | `pixel_id` + `consumer_key` | X Ads → Events Manager |

**LinkedIn note:** Use the **conversion rule ID**, not the Insight Tag ID — they are different numbers. The rule must have method set to "Conversions API" in Campaign Manager.

**Reddit note:** The pixel ID is the same as your Reddit Ads advertiser account ID (`a2_XXXXXXXXXXXX`).

**TikTok note:** The setting key is `pixel_code`, not `pixel_id`.

### Verifying a Destination

After setup, verify each destination to confirm your credentials and settings are correct:

```
POST /api/pixel/sites/{pixelId}/destinations/verify
Content-Type: application/json

{}                           // verify all enabled destinations
{ "destination_id": 10 }    // verify one specific destination
```

Verification sends a safe probe to the platform (a test event or read-only check) that never affects your real conversion counts. The result is recorded as `verified_at` on the destination — if it fails, `last_verification_error` explains why.

You can also ask the Synter agent: *"Verify my conversion tracking destinations"*.

### Event Name Mapping

Synter maps canonical pixel event names to each platform's event types:

| Pixel Event | Meta | LinkedIn | Reddit | TikTok |
|---|---|---|---|---|
| `signup` / `sign_up` | `CompleteRegistration` | `SIGN_UP` | `SIGN_UP` | `CompleteRegistration` |
| `purchase` | `Purchase` | `PURCHASE` | `PURCHASE` | `PlaceAnOrder` |
| `lead` / `demo_request` | `Lead` | `LEAD_GENERATION` | `LEAD` | `SubmitForm` |
| `trial` / `start_trial` | `StartTrial` | `SIGN_UP` | `SIGN_UP` | `Subscribe` |
| `page_view` | `PageView` | — | `PAGE_VISIT` | `ViewContent` |

Events with no mapping for a given platform are skipped silently — they don't cause errors or block other platforms.

---

## Overview

Synter provides a single API to track conversions across all your ad platforms. We automatically detect which platform the click came from and send the conversion to the right place.

## How It Works

1. User clicks an ad → arrives at your site with a click ID in the URL
2. You call `synter.conversions.create()` with the event details
3. Synter auto-detects the platform from the click ID
4. Conversion is sent to the ad platform's API
5. Optionally, event is also sent to your analytics platforms

## Click ID Detection

Each platform uses a different URL parameter:

| Platform | Parameter | Example |
|----------|-----------|---------|
| Google Ads | `gclid` | `?gclid=CjwKCAjw...` |
| Reddit | `rdt_cid` | `?rdt_cid=abc123` |
| LinkedIn | `li_fat_id` | `?li_fat_id=xyz789` |
| Microsoft | `msclkid` | `?msclkid=def456` |
| Meta | `fbclid` | `?fbclid=fb123` |
| X/Twitter | `twclid` | `?twclid=tw789` |

## Basic Usage

### TypeScript

```typescript
// Auto-detects platform from URL
await synter.conversions.create({
  event: 'purchase',
  value: 299.99,
  currency: 'USD'
});

// Or provide click ID manually
await synter.conversions.create({
  clickId: req.query.gclid,
  event: 'signup',
  value: 0
});
```

### Python

```python
# Auto-detects platform from click ID
await synter.conversions.create({
    "click_id": request.args.get("gclid"),
    "event": "purchase",
    "value": 299.99,
    "currency": "USD"
})
```

## Fan Out to Multiple Destinations

Send conversions to ad platforms AND analytics tools simultaneously:

```typescript
await synter.conversions.create({
  event: 'purchase',
  value: 299.99,
  sendTo: ['google', 'reddit', 'posthog', 'mixpanel']
});
```

## Conversion Events

Standard events we support:

| Event | Description | Typical Value |
|-------|-------------|---------------|
| `page_view` | Page viewed | 0 |
| `signup` | User signed up | 0 |
| `lead` | Lead captured | 0-50 |
| `add_to_cart` | Item added to cart | Item price |
| `begin_checkout` | Checkout started | Cart value |
| `purchase` | Purchase completed | Order value |
| `subscription` | Subscription started | MRR |

## Custom Events

You can also track custom events:

```typescript
await synter.conversions.create({
  event: 'demo_scheduled',
  value: 500,  // Estimated lead value
  properties: {
    plan: 'enterprise',
    source: 'pricing_page'
  }
});
```

## Server-Side Tracking

For server-side conversion tracking, store the click ID on your backend:

```typescript
// On page load, store click ID in session
app.get('/', (req, res) => {
  if (req.query.gclid) {
    req.session.clickId = req.query.gclid;
    req.session.clickPlatform = 'google';
  }
});

// On conversion, use stored click ID
app.post('/checkout', async (req, res) => {
  await synter.conversions.create({
    clickId: req.session.clickId,
    event: 'purchase',
    value: req.body.orderTotal
  });
});
```

## Offline Conversions

For conversions that happen offline (phone calls, in-person sales):

```typescript
await synter.conversions.create({
  clickId: storedClickId,
  event: 'offline_purchase',
  value: 5000,
  conversionTime: new Date('2025-01-15T14:30:00Z')  // When conversion actually happened
});
```

## Debugging

Use the tracking utilities to debug click detection:

```typescript
import { detectClickId, extractTrackingParams } from '@synter/sdk';

// Detect click ID from URL
const { platform, clickId } = detectClickId(window.location.href);
console.log(platform); // 'google'
console.log(clickId);  // 'CjwKCAjw...'

// Extract all tracking params
const tracking = extractTrackingParams(window.location.href, document.referrer);
console.log(tracking.clickIds);  // { google: 'abc123', reddit: 't3_xyz' }
console.log(tracking.utm);        // { source: 'google', medium: 'cpc', ... }
```

## Best Practices

1. **Store click IDs server-side** for accurate attribution
2. **Use consistent event names** across your codebase
3. **Include transaction IDs** to dedupe conversions
4. **Send conversions as soon as possible** after they occur
5. **Test with sandbox mode** before going live
