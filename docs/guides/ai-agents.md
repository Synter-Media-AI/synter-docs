# Agentic Control

Synter is built for AI agents. There are two real ways to drive it programmatically, and this guide covers both. There is **no** `synter.agents.*` API in any SDK — agentic control happens through the mechanisms below.

## Two ways in

1. **The hosted MCP server** — AI agents (Claude, Cursor, Codex, ChatGPT) connect to Synter's Model Context Protocol server and call its tools directly. This is how a conversational agent operates your ad accounts.
   Setup: [syntermedia.ai/docs/quickstart](https://syntermedia.ai/docs/quickstart)

2. **The SDK `execute()` escape hatch** — from your own backend code, `execute(scriptName, args, platform?)` runs any tool in the catalog by name. This is how you script Synter into your own automations.
   Catalog: [syntermedia.ai/docs/tools](https://syntermedia.ai/docs/tools)

Both talk to the same production endpoint (`POST https://syntermedia.ai/api/v1/tools/run`), so a call behaves identically whether an MCP agent or your code makes it.

> **⚠️ Server-side only.** Your `SYNTER_API_KEY` can spend money and modify ad accounts. Keep it on a backend, never in client-side code.

## The `execute()` escape hatch

The typed SDK methods (`campaigns.*`, `analytics.*`, `conversions.*`, `keywords.*`, `creative.*`, `audiences.*`, ...) cover the ~25 most common tools. `execute()` reaches any of the 140+ backend scripts beyond them.

### TypeScript

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY!);

// args is an idiomatic { flagName: value } map, converted to CLI flags internally
await synter.execute('google_ads_list_audiences', { status: 'ENABLED' }, 'google');
```

### Python

```python
from synter import Synter

client = Synter(api_key="syn_...")

client.execute("google_ads_list_audiences", {"status": "ENABLED"}, "google")
```

### Rust

```rust
use std::collections::HashMap;
use serde_json::json;

let mut args = HashMap::new();
args.insert("status".to_string(), json!("ENABLED"));
client.execute("google_ads_list_audiences", args, Some("google")).await?;
```

## Optimization is composed from real tools

Budget and bid changes are not run by a magic optimizer object — you compose them from the tools that actually exist. Read performance, decide, then act:

```typescript
// 1. Read performance
const perf = await synter.analytics.getPerformance({ date_range: 'LAST_30_DAYS' });

// 2. Act on it with typed methods...
await synter.campaigns.updateBudget({ campaign_id: '123', platform: 'google', daily_budget: 75 });
await synter.campaigns.pause({ campaign_id: '456', platform: 'google' });

// ...or reach any other backend script via execute()
await synter.execute('<some_optimization_script>', { /* flag: value */ }, 'google');
```

Keep your own guardrails (max budget change, excluded campaigns, approval thresholds) in your application logic around these calls.

## The MCP dry-run safety model

When an AI agent drives Synter through the MCP server's action runner, every action is a **validation-only dry run unless the agent explicitly opts in**:

```json
{ "action": "reddit_ads_create_post", "args": ["--headline", "..."], "dry_run": false }
```

- `dry_run` defaults to `true`: the action is whitelisted, its arguments are validated, and credentials and credit cost are resolved — but **nothing runs**.
- The dry-run response tells the agent how to proceed (`"next_step": "Re-call with dry_run=false to actually run this action."`), so agents self-discover the protocol at runtime.
- Pass `dry_run: false` only after the dry run validates and, for anything that spends money, only with the account owner's approval.
- Many scripts also accept their own `--dry-run` flag in `args` for a richer script-level preview.

## Best Practices

1. **Preview before you spend** — rely on the MCP dry-run default, or a script's own `--dry-run` flag.
2. **Read before you write** — pull `analytics.getPerformance` before changing budgets or bids.
3. **Keep guardrails in your code** — caps, exclusions, and approval thresholds live in your automation, not in a hidden agent.
4. **Protect critical campaigns** — never auto-touch brand or retargeting campaigns.

## Reference

- MCP setup: [syntermedia.ai/docs/quickstart](https://syntermedia.ai/docs/quickstart)
- Tool catalog: [syntermedia.ai/docs/tools](https://syntermedia.ai/docs/tools)
- [TypeScript SDK](../../sdk/typescript/README.md) · [Python SDK](../../sdk/python/README.md) · [Rust SDK](../../sdk/rust/README.md)
