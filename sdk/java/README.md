# Synter Java SDK

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. A typed Java client for managing Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, and X ad campaigns from one client.

> **`0.1.0` — live on Maven Central** as `ai.syntermedia:synter-sdk`. Pre-1.0, so the surface may change before a stable `1.0`; pin a version in production. Requires Java 17+.

## Requirements

- Java 17+
- A Synter API key — get one at [syntermedia.ai/developer](https://syntermedia.ai/developer) (format: `syn_` followed by 32 base64url characters)

## Installing (planned)

From Maven Central:

```kotlin
// build.gradle.kts
dependencies {
    implementation("ai.syntermedia:synter-sdk:0.1.0")
}
```

```xml
<!-- pom.xml -->
<dependency>
  <groupId>ai.syntermedia</groupId>
  <artifactId>synter-sdk</artifactId>
  <version>0.1.0</version>
</dependency>
```

## Quick Start (planned)

```java
import ai.syntermedia.sdk.Synter;
import ai.syntermedia.sdk.requests.ListCampaignsRequest;
import ai.syntermedia.sdk.requests.CreateSearchCampaignRequest;

import java.util.List;
import java.util.Map;

Synter synter = Synter.builder()
    .apiKey(System.getenv("SYNTER_API_KEY"))
    .build();

// List campaigns (defaults to Google if no platform given)
Map<String, Object> campaigns = synter.campaigns().list(ListCampaignsRequest.builder()
    .platform("google")
    .status("ENABLED")
    .build());

// Create a Google Ads Search campaign
Map<String, Object> result = synter.campaigns().createSearch(CreateSearchCampaignRequest.builder()
    .campaignName("Q4 Launch")
    .dailyBudget(50.0)
    .finalUrl("https://example.com")
    .keywords(List.of("running shoes"))
    .headlines(List.of("Buy Running Shoes", "Free Shipping Today", "Shop The Collection"))
    .descriptions(List.of("Premium running shoes for every terrain.", "Order today, ships free."))
    .build());
```

> **⚠️ Server-side only.** Never hardcode the key — read it from an environment variable or secrets manager (`System.getenv("SYNTER_API_KEY")`). The key can spend money and modify ad accounts, so keep it out of client-side code.

## Method reference (planned)

| Resource | Methods |
|---|---|
| `synter.campaigns()` | `list`, `createSearch`, `createDisplay`, `createPmax`, `pause`, `updateBudget` |
| `synter.analytics()` | `getPerformance`, `getDailySpend` |
| `synter.keywords()` | `add`, `addNegative` |
| `synter.conversions()` | `create`, `list`, `diagnoseTracking` |
| `synter.creative()` | `generateImage`, `generateVideo` |
| `synter.meta()` | `createCampaign` |
| `synter.linkedin()` | `createCampaign` |
| `synter.reddit()` | `createCampaign` |
| `synter.audiences()` | `stageArtifact`, `sync`, `manage` |
| top-level on `synter` | `listAdAccounts()`, `uploadImage(...)`, `listLandingPages()`, `execute(...)` |

`synter.execute(scriptName, args, platform)` is the universal escape hatch — it can run any of the 140+ backend scripts by name, not just the ~25 covered by typed methods above. `args` is a plain `Map<String, Object>` of flag-name to value (e.g. `Map.of("status", "ENABLED")`).

## While you wait

- Watch this repo for the Maven Central release.
- Use the live [TypeScript](../typescript/README.md), [Python](../python/README.md), or [Rust](../rust/README.md) SDKs, or call the API directly (`POST https://syntermedia.ai/api/v1/tools/run`).
- Full references: [syntermedia.ai/docs/sdks](https://syntermedia.ai/docs/sdks) and [syntermedia.ai/docs/api](https://syntermedia.ai/docs/api).

## License

MIT
