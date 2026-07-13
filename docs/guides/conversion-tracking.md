# Conversion Tracking

Get conversions the ad platforms cannot see back into the ad platforms, so your ROAS reflects reality. This is the highest-value thing to automate over the REST API: it is deterministic, high-volume, and should run server-side without an agent in the loop.

## The problem this solves

Ad platforms only count what happens in a self-serve checkout. Offline closes, phone quotes, CRM-won deals, and high-consideration purchases are invisible to them, so your best-performing campaigns can look like your worst. The fix is to upload those conversions back to each platform, keyed to the original click ID, so the platform can attribute and optimize correctly.

## How to send conversions

Use the REST endpoint `POST https://syntermedia.ai/api/v1/tools/run` with `Authorization: Bearer syn_...`. Each platform has a conversion tool. To see the ones available to your key:

```bash
curl https://syntermedia.ai/api/v1/tools/run \
  -H "Authorization: Bearer $SYNTER_API_KEY"
```

### Google Ads: offline conversions (CSV batch)

Best for batch upload from a CRM export or warehouse query. `google_ads_upload_offline_conversions` accepts a CSV by URL, by inline data, or from a file. Each row carries the click ID (gclid), the conversion time, and the value.

```bash
# Validate first with --dry-run (no upload, no platform write)
curl -X POST https://syntermedia.ai/api/v1/tools/run \
  -H "Authorization: Bearer $SYNTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "script_name": "google_ads_upload_offline_conversions",
    "platform": "google",
    "customer_id": "1234567890",
    "args": ["--csv-url", "https://yourapp.com/exports/conversions.csv", "--conversion-name", "Offline Purchase", "--dry-run"]
  }'
```

Remove `--dry-run` to upload for real. Useful flags: `--csv-url`, `--csv-data` (inline CSV text), `--conversion-name`, `--currency`, `--timezone`, and `--validate-only` (send to Google Ads as a validation-only request). Run the tool with no CSV to see the required columns in the error, or check the dry-run output.

### Reddit: single-event conversions (CAPI)

`reddit_ads_send_conversion` sends one event at a time, keyed to the Reddit click ID (`rdt_cid`) or a hashed email:

```bash
curl -X POST https://syntermedia.ai/api/v1/tools/run \
  -H "Authorization: Bearer $SYNTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "script_name": "reddit_ads_send_conversion",
    "platform": "reddit",
    "args": ["--event-type", "Purchase", "--click-id", "RDT_CID", "--value", "5000", "--currency", "USD"]
  }'
```

Add `--test-mode` to send it as a test event while you wire things up.

More platforms (Meta, LinkedIn, TikTok, X) expose their own conversion tools. The list endpoint above is the source of truth for what your workspace can run today.

## Click ID reference

Capture and store the click ID when the visitor lands, then attach it to the conversion when it closes.

| Platform | URL parameter |
|----------|---------------|
| Google Ads | `gclid` |
| Reddit | `rdt_cid` |
| LinkedIn | `li_fat_id` |
| Microsoft | `msclkid` |
| Meta | `fbclid` |
| X/Twitter | `twclid` |
| TikTok | `ttclid` |

## Server-side pattern

For offline or delayed conversions, store the click ID at landing time and send the conversion when the deal actually closes:

```typescript
// At landing: persist the click ID against the lead/session
app.get("/", (req, res) => {
  if (req.query.gclid) saveLead({ gclid: req.query.gclid, platform: "google" });
});

// On close (could be days later): export won deals to a CSV your app serves,
// then run the batch upload tool over the REST API on a schedule.
await runTool({
  script_name: "google_ads_upload_offline_conversions",
  platform: "google",
  customer_id: "1234567890",
  args: ["--csv-url", "https://yourapp.com/exports/won-deals.csv", "--conversion-name", "CRM Won"],
});
```

`runTool` is the small `fetch` wrapper from the [Node REST guide](../../sdk/typescript/README.md). A Python version is in the [Python REST guide](../../sdk/python/README.md).

## Best practices

1. Store click IDs server-side the moment a visitor lands, so you still have them when a deal closes weeks later.
2. Include the real conversion time so platform attribution windows line up.
3. Always run a `--dry-run` (or `--validate-only`) pass before a live batch.
4. Send a stable transaction/order ID per row so re-runs deduplicate instead of double-counting.
5. Schedule the upload (hourly or daily) rather than firing one call per event, to keep credit usage low.
