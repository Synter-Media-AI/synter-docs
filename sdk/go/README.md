# synter-go

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. Manage Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, and X Ads campaigns from Go code or an AI agent.

> **🚧 Coming soon — not yet published.** The Go SDK is built but has not been tagged or published to the Go module proxy yet. The install command and import path below are the planned surface; they are not resolvable until the first release. For a language that is live today, see the [TypeScript](../typescript/README.md), [Python](../python/README.md), or [Rust](../rust/README.md) SDKs.

## Installing (planned)

```bash
go get github.com/Synter-Media-AI/synter-go
```

## Getting an API key

Get a `syn_...` API key at [syntermedia.ai/developer](https://syntermedia.ai/developer).

> **⚠️ Server-side only.** The key is a secret that can spend money and modify ad accounts. Use this SDK from a backend, never ship it in client code.

## Quick Start (planned)

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
		FinalURL:     "https://example.com/shoes",
	})
	if err != nil {
		log.Fatal(err)
	}
}
```

## Method reference (planned)

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

`client.Execute(ctx, scriptName, args, platform)` is the universal escape hatch — it can run any of the 140+ backend scripts by name, not just the ~25 covered by typed methods above.

## While you wait

- Watch this repo for the tagged release.
- Use the live [TypeScript](../typescript/README.md), [Python](../python/README.md), or [Rust](../rust/README.md) SDKs, or call the API directly (`POST https://syntermedia.ai/api/v1/tools/run`).
- Full references: [syntermedia.ai/docs/sdks](https://syntermedia.ai/docs/sdks) and [syntermedia.ai/docs/api](https://syntermedia.ai/docs/api).

## License

MIT
