# AI Agents

Unlike enterprise black-box platforms, Synter gives you full visibility into and control over AI optimization.

## Overview

Synter's agent model is:
- **Transparent**: See exactly what changes are proposed and why
- **Controllable**: Every action validates in dry-run mode first; you review, then apply
- **Auditable**: Full history of all agent runs and actions
- **Approval-gated**: Nothing that spends money runs without the account owner's approval

## How Agents Connect

Agents (Claude, Cursor, custom Claude Agent SDK apps, or anything MCP-compatible) connect through the Synter MCP server:

```json
{
  "mcpServers": {
    "synter": {
      "type": "http",
      "url": "https://mcp.syntermedia.ai",
      "headers": { "X-Synter-Key": "YOUR_API_KEY" }
    }
  }
}
```

From there they get the full toolset: campaign reads and writes, creative generation, audiences, attribution, budget optimization, and GA4 (free, no credits).

## MCP `execute` is Dry-Run by Default

The MCP `execute` tool is the universal action runner used by agents connected to Synter. Every `execute` call is a **validation-only dry run unless you explicitly opt in**:

```json
{ "action": "reddit_ads_create_post", "args": ["--headline", "..."], "dry_run": false }
```

- `dry_run` defaults to `true`: the action is whitelisted, its arguments are validated, credentials and credit cost are resolved, and **nothing runs**.
- The dry-run response tells the agent exactly how to proceed: `"next_step": "Re-call execute with dry_run=false to actually run this action."` Agents self-discover the protocol at runtime even if they never read this page.
- Pass `dry_run: false` only after the dry run validates and, for anything that spends money, only with the account owner's approval.
- Tip for script-level previews: many scripts accept their own `--dry-run` flag in `args`. When you pass it, the script itself simulates and returns a richer preview than the top-level gate.

## What Agents Can Do

| Capability | Example tools |
|-----------|---------------|
| Budget optimization | `optimize_budget` — reallocate spend by ROAS across platforms |
| Campaign control | `pause_campaign`, `enable_campaign`, `update_campaign_budget` |
| Forecasting | `forecast_campaign` — project outcomes before spending |
| Creative | `generate_image_ad`, `generate_video_ad`, `test_creatives` |
| Audiences | `build_abm_audience`, `build_lookalike_audience`, `sync_audience` |
| Measurement | `get_attribution`, `measure_incrementality`, `reconcile_platforms` |

## Autonomous Schedules

For recurring optimization, agents can register autonomous schedules that run on a cadence:

- `create_autonomous_schedule` — set up a recurring optimization run
- `list_autonomous_schedules` — review what is scheduled

Scheduled runs follow the same safety model: proposed changes are logged, and spend-affecting actions respect your account's approval settings.

## Guardrails & Safety

- **Approval before spend** — campaign launches, budget increases, and other money-moving actions require explicit approval from the account owner.
- **Dry-run everywhere** — every write validates before it executes.
- **Spend alerts** — set alerts (`set_spend_alert`) so runaway spend pages you, not your credit card statement.
- **Read operations are always safe** — pulling metrics and listing campaigns never changes anything.

## Best Practices

1. **Start with dry-run** — always preview changes before applying
2. **Start with reads** — let an agent report on performance before letting it act
3. **Protect critical campaigns** — tell your agent which brand/retargeting campaigns are off-limits
4. **Review agent history** — check what ran and what changed weekly
5. **Automate gradually** — move to scheduled autonomy only after building trust
