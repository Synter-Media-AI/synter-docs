# AI Agents

Unlike enterprise black-box platforms, Synter's agents are transparent and controllable, and nothing that spends money runs without approval.

## Overview

Synter's agents are:
- **Transparent**: you see exactly what they propose and why before anything changes
- **Controllable**: every action validates as a dry run first, then runs only when you opt in
- **Auditable**: full history of runs and actions in your workspace
- **Approval-gated**: writes that spend money ask for explicit approval

## How to run agents

There are three ways, all backed by the same tools and the same approval model:

1. **In the Synter product** — chat with the agent in your workspace, or set up an Autonomous Operator schedule to monitor and adjust campaigns on a cadence.
2. **Through the MCP** — connect Claude, Cursor, or Codex to `https://mcp.syntermedia.ai` and drive the agent in natural language. See the [Claude Plugin & MCP guide](./claude-plugin.md). The packaged plugin also ships named subagents: `campaign-strategist`, `media-buyer`, `audience-builder`, `creative-director`, `budget-optimizer`, `performance-analyst`.
3. **Over the REST API** — run individual tools deterministically with `POST /api/v1/tools/run`. Use this for scheduled optimization or reporting jobs where no human is in the loop. See the [REST API guide](../../sdk/typescript/README.md).

## The approval-before-spend model

Whether an action arrives through the MCP `execute` tool or the REST endpoint, the same safety model applies: **every action validates first and runs only when you explicitly opt in.**

Through the MCP `execute` tool (the universal action runner used by Claude, Codex, and other connected agents):

```json
{ "action": "reddit_ads_create_post", "args": ["--headline", "..."], "dry_run": false }
```

- `dry_run` defaults to `true`: the action is whitelisted, its arguments are validated, credentials and credit cost are resolved, and nothing runs.
- The dry-run response tells the agent how to proceed: `"next_step": "Re-call execute with dry_run=false to actually run this action."` Agents self-discover the protocol at runtime even if they never read this page.
- Pass `dry_run: false` only after the dry run validates, and for anything that spends money, only with the account owner's approval.
- Many scripts also accept their own `--dry-run` flag in `args`. When you pass it, the script simulates and returns a richer preview than the top-level gate.

Over the REST API, the equivalent is to pass a tool's own `--dry-run` (or `--validate-only`) flag in `args` first, confirm the preview, then re-run without it.

## What the agents do

| Capability | What it does |
|------------|--------------|
| Budget optimization | Reallocates budget from underperformers to top performers, within limits you set |
| Bid optimization | Adjusts bids toward a target CAC or ROAS |
| Conversion upload | Syncs offline and delayed conversions back to the ad platforms (see [Conversion Tracking](./conversion-tracking.md)) |
| Audience building | Builds ABM lists, lookalikes, and signal-based segments |
| Creative generation | Produces on-brand images, video, and ad copy |
| Reporting | Cross-channel performance reports and executive summaries |

## Guardrails

Agents operate inside the limits configured on your workspace: budget caps, per-change ceilings, protected campaigns that are never touched, and approval thresholds for large changes. Reads are always free to run. Configure these in the Synter product, or ask the agent to set them for you.

## Best practices

1. Preview first. Let a dry run validate before you approve a real change.
2. Start conservative. Use small per-change limits and widen them as you build trust.
3. Protect brand and retargeting campaigns with an exclusion list.
4. Review run history weekly.
5. Move to autonomous schedules only after the manual runs look right.
