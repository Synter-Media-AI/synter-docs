# synter (Rust)

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. A cross-platform advertising client for Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, and X — built for AI agents and developers.

> **`0.1.1` — live on crates.io.** Pre-1.0, so the surface may change before a stable `1.0`; pin a version in production. `0.1.1` fixes `analytics().get_performance(..)` for `Platform::Google` and `Platform::Linkedin`, which in `0.1.0` sent the script filename (`pull_google_ads_data`) instead of the canonical script name (`pull_google_ads`).

## Installing

```bash
cargo add synter
```

or in `Cargo.toml`:

```toml
[dependencies]
synter = "0.1"
```

> **⚠️ Server-side only.** The key is a secret that can spend money and modify ad accounts. Use this SDK from a backend, never ship it in client code.

## Getting an API key

1. Go to [synterai.com/developer](https://synterai.com/developer) and generate an API key (format: `syn_` followed by 32 base64url characters).
2. Set it as an environment variable, e.g. `SYNTER_API_KEY`, or pass it directly to the client builder.

## Quick Start

```rust
use synter::{Client, CreateSearchCampaignRequest};

#[tokio::main]
async fn main() -> Result<(), synter::SynterError> {
    let client = Client::builder()
        .api_key(std::env::var("SYNTER_API_KEY").expect("SYNTER_API_KEY not set"))
        .build()?;

    // List campaigns (defaults to Google Ads if no platform filter is given).
    let campaigns = client.campaigns().list(Default::default()).await?;
    println!("{campaigns}");

    // Create a Search campaign.
    let result = client
        .campaigns()
        .create_search(
            CreateSearchCampaignRequest::builder()
                .campaign_name("Q4 Launch")
                .daily_budget(50.0)
                .final_url("https://example.com/landing")
                .keywords(vec!["project management software".to_string()])
                .headlines(vec![
                    "Ship Projects Faster".to_string(),
                    "Try It Free Today".to_string(),
                    "Loved By 10k Teams".to_string(),
                ])
                .descriptions(vec![
                    "The project tool your team will actually use.".to_string(),
                    "Get started in minutes. No credit card required.".to_string(),
                ])
                .build(),
        )
        .await?;
    println!("{result}");

    Ok(())
}
```

## Resource namespaces

Every method mirrors one of the 25 tools in the shared tool catalog, grouped by category:

| Namespace | Methods |
|---|---|
| `client.campaigns()` | `list`, `create_search`, `create_display`, `create_pmax`, `pause`, `update_budget` |
| `client.analytics()` | `get_performance`, `get_daily_spend` |
| `client.keywords()` | `add`, `add_negative` |
| `client.conversions()` | `create`, `list`, `diagnose_tracking` |
| `client.creative()` | `generate_image`, `generate_video` |
| `client.meta()` / `client.linkedin()` / `client.reddit()` | `create_campaign` |
| `client.audiences()` | `stage_artifact`, `sync`, `manage` |
| `client` (top level) | `list_ad_accounts`, `upload_image`, `list_landing_pages`, `execute` |

`client.execute(script_name, args, platform)` is the universal escape hatch — it mirrors the MCP server's `run_tool`, letting you call any of the 140+ backend scripts beyond the 25 typed methods above. `args` is a `HashMap<String, serde_json::Value>` of flag-name -> value (e.g. `{"status": "ENABLED"}`); the SDK converts it to the backend's `--flag value` wire form internally.

For the full method reference, see [docs.synterai.com/sdk](https://docs.synterai.com/sdk) and [docs.synterai.com/api](https://docs.synterai.com/api/authentication).

## Error handling

Every fallible call returns `Result<serde_json::Value, SynterError>`. `SynterError` has three variants:

- `SynterError::Validation(String)` — a required field was missing. Raised before any network call.
- `SynterError::Api { status, message, code, details }` — the backend returned a non-2xx response.
- `SynterError::Network(reqwest::Error)` — the request never completed (DNS/TCP/TLS/timeout, or an undecodable response body).

## Retries

The client retries up to 3 times with exponential backoff (1s base, 10s cap) on `429` responses (honoring `Retry-After` when present) and on network/timeout errors. Other 4xx responses are never retried. Configurable via `ClientBuilder::max_retries`, `retry_base_delay`, and `retry_max_delay`.

## Known issues (preserved on purpose)

These mirror real, current production behavior of the Synter backend — they are not bugs in this SDK, and this SDK intentionally does not "fix" them (doing so would make calls fail against the real API):

1. `campaigns().pause(...)` and `campaigns().update_budget(...)` require a `platform` argument (matching the tool's schema, which advertises support for all 7 platforms), but the live dispatch always hardcodes `platform=google` regardless of what's passed.
2. `Client::upload_image(...)` accepts an optional platform, but the live dispatch always hardcodes `platform=google`.
3. `campaigns().create_display(...)` and `campaigns().create_pmax(...)` use inconsistent flag names for the same concept: `--landscape-image` / `--square-image` vs. `--landscape-image-url` / `--square-image-url`.

## Links

- [Documentation](https://docs.synterai.com/sdk)
- [Quick Start](../../docs/quickstart.md)
- [Guides](../../docs/guides/README.md)

## Write paths and approvals

Copy-paste examples for pausing a campaign and changing a daily budget, and how a held write comes back (HTTP 202 `pending_review` with an `audit_id`), are on [docs.synterai.com/sdk/rust](https://docs.synterai.com/sdk/rust). Which writes run immediately and which wait for a person on each surface: [Approval model](https://docs.synterai.com/mcp/approval-model).

**Known issue (0.1.x):** `updateBudget`/`update_budget` sends `--budget`, which the API rejects with HTTP 400. The fix is in the SDK source and ships in the next release; until then use the generic `execute` call shown in the docs page above.

## License

MIT
