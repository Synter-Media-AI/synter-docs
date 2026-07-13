# UTM Management

When Synter builds campaigns for you, it fills in platform-specific UTM macros automatically so your landing-page URLs carry consistent, granular attribution. This guide is the reference for how those macros map per platform.

## The problem

Each ad platform has its own dynamic-parameter syntax, and managing them by hand across platforms is error-prone:

| Platform | Example macros |
|----------|----------------|
| Google Ads | `{creative}`, `{keyword}`, `{campaignid}` |
| Reddit | `{{AD_ID}}`, `{{CAMPAIGN_ID}}` |
| LinkedIn | `{creative}`, `{campaign_id}` |
| Microsoft | `{AdId}`, `{keyword}` |

## How Synter handles it

Synter uses one universal template and translates it to each platform's native macro when it creates the campaign. You express intent once:

```
utm_source=google
utm_medium=cpc
utm_campaign=product-launch
utm_content={{ad_id}}     -> becomes {creative} on Google, {{AD_ID}} on Reddit
utm_term={{keyword}}      -> becomes {keyword} on Google/Microsoft
```

and the resulting landing-page URL on Google Ads carries:

```
?utm_source=google&utm_medium=cpc&utm_campaign=product-launch&utm_content={creative}&utm_term={keyword}
```

## Universal macro reference

| Universal variable | Google | Reddit | LinkedIn | Microsoft |
|--------------------|--------|--------|----------|-----------|
| `{{ad_id}}` | `{creative}` | `{{AD_ID}}` | `{creative}` | `{AdId}` |
| `{{campaign_id}}` | `{campaignid}` | `{{CAMPAIGN_ID}}` | `{campaign_id}` | `{CampaignId}` |
| `{{keyword}}` | `{keyword}` | N/A | N/A | `{keyword}` |
| `{{placement}}` | `{placement}` | `{{SUBREDDIT}}` | N/A | `{placement}` |
| `{{device}}` | `{device}` | `{{DEVICE}}` | N/A | `{device}` |

## Best practices

1. Use consistent campaign names across platforms for easier cross-platform analysis.
2. Include `ad_id` and `campaign_id` in UTMs for granular attribution.
3. Use `term` for keywords on search platforms (Google, Microsoft).
4. Use `content` for creative variants to A/B test ad copy.
