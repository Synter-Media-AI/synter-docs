# AI Agents

Unlike enterprise black-box platforms, Synter gives you full programmatic control over AI optimization agents.

## Overview

Synter's AI agents are:
- **Transparent**: See exactly what changes they propose and why
- **Controllable**: Run in dry-run mode, review changes, then apply
- **Auditable**: Full history of all agent runs and actions
- **Configurable**: Set guardrails, limits, and preferences

## Available Agents

| Agent | Description |
|-------|-------------|
| `budget-optimizer` | Reallocates budget across campaigns based on performance |
| `bid-optimizer` | Adjusts bids to hit target CAC/ROAS |
| `conversion-uploader` | Syncs offline conversions to ad platforms |
| `audience-expander` | Suggests new audiences based on top performers |
| `creative-analyzer` | Analyzes ad creative performance patterns |

## Running Agents

### Dry-Run Mode (Recommended)

Always start with dry-run to preview changes:

```typescript
const optimization = await synter.agents.run('budget-optimizer', {
  dryRun: true,
  window: { start: '2025-11-01', end: '2025-11-08' },
  params: {
    maxBudgetChange: 0.15,  // ±15%
    minConversions: 10
  }
});

// Review proposed changes
console.log(optimization.result.proposedChanges);
// [
//   { campaignId: 'cmp_123', currentBudget: 500, proposedBudget: 575, reason: 'CAC below target' },
//   { campaignId: 'cmp_456', currentBudget: 300, proposedBudget: 240, reason: 'CAC above threshold' }
// ]
```

### Apply Changes

After reviewing, apply the changes:

```typescript
if (userApproved) {
  await synter.agents.apply(optimization.runId);
}
```

### MCP `execute` is Dry-Run by Default

The same safety model applies to the MCP `execute` tool (the universal action runner used by Claude, Codex, and other agents connected to Synter). Every `execute` call is a **validation-only dry run unless you explicitly opt in**:

```json
{ "action": "reddit_ads_create_post", "args": ["--headline", "..."], "dry_run": false }
```

- `dry_run` defaults to `true`: the action is whitelisted, its arguments are validated, credentials and credit cost are resolved, and **nothing runs**.
- The dry-run response tells the agent exactly how to proceed: `"next_step": "Re-call execute with dry_run=false to actually run this action."` Agents self-discover the protocol at runtime even if they never read this page.
- Pass `dry_run: false` only after the dry run validates and, for anything that spends money, only with the account owner's approval.
- Tip for script-level previews: many scripts accept their own `--dry-run` flag in `args`. When you pass it, the script itself simulates and returns a richer preview than the top-level gate.

### Auto-Pilot Mode

For trusted agents, run without dry-run:

```typescript
await synter.agents.run('conversion-uploader', {
  dryRun: false,
  platforms: ['google', 'reddit']
});
```

## Agent Configuration

Configure agent behavior globally or per-run:

```typescript
// Update global config
await synter.agents.updateConfig('budget-optimizer', {
  maxBudgetChange: 0.20,      // ±20%
  minConversions: 5,
  targetCAC: 50,
  excludeCampaigns: ['cmp_brand']  // Never touch brand campaigns
});

// Override per-run
await synter.agents.run('budget-optimizer', {
  params: {
    maxBudgetChange: 0.10  // More conservative for this run
  }
});
```

## Agent History

View past runs and their outcomes:

```typescript
const runs = await synter.agents.runs('budget-optimizer', {
  limit: 10,
  startDate: '2025-11-01'
});

for (const run of runs) {
  console.log(`${run.id}: ${run.status} - ${run.changesApplied} changes`);
}
```

## Budget Optimizer

The most commonly used agent. Reallocates budget from underperforming campaigns to top performers.

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `maxBudgetChange` | number | 0.15 | Max % change per campaign (0.15 = ±15%) |
| `minConversions` | number | 10 | Min conversions to consider a campaign |
| `targetCAC` | number | auto | Target CAC (uses account average if not set) |
| `targetROAS` | number | auto | Target ROAS (alternative to CAC) |
| `excludeCampaigns` | string[] | [] | Campaign IDs to never modify |
| `platforms` | string[] | all | Which platforms to optimize |

### Example

```typescript
const result = await synter.agents.run('budget-optimizer', {
  dryRun: true,
  params: {
    maxBudgetChange: 0.20,
    minConversions: 5,
    targetCAC: 45,
    excludeCampaigns: ['cmp_brand', 'cmp_retargeting']
  }
});
```

## Bid Optimizer

Adjusts keyword/audience bids to hit target metrics.

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `targetCAC` | number | required | Target cost per acquisition |
| `maxBidChange` | number | 0.25 | Max % change per bid |
| `minClicks` | number | 20 | Min clicks to consider |
| `bidFloor` | number | 0.10 | Minimum bid |
| `bidCeiling` | number | 50.00 | Maximum bid |

## Conversion Uploader

Syncs offline/delayed conversions to ad platforms.

```typescript
await synter.agents.run('conversion-uploader', {
  platforms: ['google', 'reddit'],
  lookbackDays: 30,  // Upload conversions from last 30 days
  dryRun: false
});
```

## Guardrails & Safety

### Budget Caps

Hard limits that agents will never exceed:

```typescript
await synter.agents.updateConfig('budget-optimizer', {
  hardCaps: {
    maxDailyBudget: 1000,      // Never set daily budget above $1000
    maxTotalBudget: 10000,     // Never exceed $10k total daily spend
    minCampaignBudget: 10      // Never drop below $10/day
  }
});
```

### Approval Requirements

Require human approval for large changes:

```typescript
await synter.agents.updateConfig('budget-optimizer', {
  requireApprovalIf: {
    budgetChangePercent: 0.30,  // Changes > 30%
    budgetChangeAbsolute: 500,  // Changes > $500
    affectedCampaigns: 5        // Touching > 5 campaigns
  }
});
```

### Notifications

Get notified of agent actions:

```typescript
await synter.agents.updateConfig('budget-optimizer', {
  notifications: {
    onRun: ['slack:#marketing'],
    onApply: ['email:team@company.com'],
    onError: ['slack:#alerts', 'email:ops@company.com']
  }
});
```

## Best Practices

1. **Start with dry-run** - Always preview changes before applying
2. **Set conservative limits** - Start with low `maxBudgetChange` and increase over time
3. **Exclude critical campaigns** - Protect brand and retargeting campaigns
4. **Monitor performance** - Review agent history weekly
5. **Use guardrails** - Set hard caps and approval requirements
6. **Automate gradually** - Move to auto-pilot only after building trust
