# synter-go

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. Manage **16 direct ad platforms** (Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, X, Pinterest, Snapchat, Spotify, Amazon Ads, Amazon DSP, The Trade Desk, OpenAI Ads, DV360, StackAdapt) and **43 total platforms** from Go code or an AI agent.

## Installing

```bash
go get github.com/Synter-Media-AI/synter-go
```

## Getting an API key

Get a `syn_...` API key at [synterai.com/developer](https://synterai.com/developer).

> **⚠️ Server-side only.** The key is a secret that can spend money and modify ad accounts. Use this SDK from a backend, never ship it in client code.

## Quick Start

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"

	synter "github.com/Synter-Media-AI/synter-go"
)

func main() {
	client := synter.NewClient(os.Getenv("SYNTER_API_KEY"))
	ctx := context.Background()

	// List campaigns (defaults to Google if no platform is given)
	campaigns, err := client.Campaigns.List(ctx, synter.ListCampaignsRequest{
		Platform: "google",
		Status:   "ENABLED",
	})
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(campaigns)

	// Create a Google Search campaign
	_, err = client.Campaigns.CreateSearch(ctx, synter.CreateSearchCampaignRequest{
		CampaignName: "Q4 Launch",
		DailyBudget:  50,
		Keywords:     []string{"running shoes"},
		Headlines:    []string{"Fast Shoes", "Buy Now", "Free Shipping"},
		Descriptions: []string{"Great shoes.", "Ships in 2 days."},
		FinalURL:     "https://synterai.com/get-started",
	})
	if err != nil {
		log.Fatal(err)
	}
}
```

## Method reference

| Namespace | Methods |
|---|---|
| `client.Campaigns` | `List`, `CreateSearch`, `CreateDisplay`, `CreatePmax`, `Pause`, `UpdateBudget` |
| `client.Analytics` | `GetPerformance`, `GetDailySpend` |
| `client.Keywords` | `Add`, `AddNegative` |
| `client.Conversions` | `Create`, `List`, `DiagnoseTracking` |
| `client.Creative` | `GenerateImage`, `GenerateVideo` |
| `client.Meta` | `CreateCampaign` |
| `client.LinkedIn` | `CreateCampaign` |
| `client.Reddit` | `CreateCampaign` |
| `client.Audiences` | `StageArtifact`, `Sync`, `Manage` |
| `client` (top-level) | `ListAdAccounts`, `UploadImage`, `ListLandingPages`, `Execute` |

`client.Execute(ctx, scriptName, args, platform)` is the universal escape hatch — it can run any of the 155+ backend scripts by name, not just the 25 covered by typed methods above.

## See also

- The other SDKs: [TypeScript](../typescript/README.md), [Python](../python/README.md), [Rust](../rust/README.md), [Java](../java/README.md).
- Call the API directly (`POST https://synterai.com/api/v1/tools/run`).
- Full references: [docs.synterai.com/sdk](https://docs.synterai.com/sdk) and [docs.synterai.com/api](https://docs.synterai.com/api).

## License

MIT
