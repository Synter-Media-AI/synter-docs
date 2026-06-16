# Claude Plugin

Run Synter inside Claude — the AI Agent Operator for Ads, packaged as a Claude Code / Claude Desktop plugin. Connect every ad platform, build audiences, generate on-brand creative, launch cross-platform campaigns, and reallocate spend by ROAS, all in one conversation. You direct, the agents execute. Nothing that spends money ships without your approval.

**Repository:** [github.com/Synter-Media-AI/plugin](https://github.com/Synter-Media-AI/plugin)

## Install

In Claude Code:

```text
/plugin marketplace add Synter-Media-AI/plugin
/plugin install synter@synter
```

When enabling, paste your Synter API key (`syn_...`) — or leave it blank and run `/synter:quickstart` to onboard and get one. Create a key at [syntermedia.ai/developer](https://syntermedia.ai/developer).

Free GA4 and onboarding tools work with no key and no credits.

> Requires Claude Code with plugin support. Campaign write actions spend real money — every spend asks for your approval first.

## What it packages

| Component | What you get |
|-----------|--------------|
| **MCP server** | The Synter Advertising Platform MCP — cross-platform campaign read/write, creative generation, audiences, attribution, and GA4. Tools register automatically once the plugin is enabled. |
| **Skills** (`/synter:*`) | Guided workflows you or Claude can invoke. |
| **Agents** | Specialized subagents Claude dispatches when the task fits. |
| **Safety hook** | Enforces approval-before-spend and brand voice on every session. |

### Skills

| Command | Does |
|---------|------|
| `/synter:quickstart` | Onboard, connect a platform, run a first action. |
| `/synter:connect` | Link ad platforms and GA4 — Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, X, Amazon, and more. |
| `/synter:audience` | Build ABM lists, lookalikes, and signal-based segments; activate them. |
| `/synter:creative` | Generate on-brand images, video, UGC, and ad copy. |
| `/synter:launch` | Plan, preflight, and ship a cross-platform campaign. |
| `/synter:optimize` | Cut wasted spend, scale winners, reallocate budget by ROAS. |
| `/synter:report` | Cross-channel performance report and exec summary. |
| `/synter:help` | What Synter can do, and where to get support. |

### Agents

`campaign-strategist` · `media-buyer` · `audience-builder` · `creative-director` · `budget-optimizer` · `performance-analyst`

They run automatically when the task fits, or you can call one directly from `/agents`.

## Run it headlessly (Claude Agent SDK)

The repository ships a runner that loads the plugin with the [Claude Agent SDK](https://www.npmjs.com/package/@anthropic-ai/claude-agent-sdk) and runs the operator unattended — for cron digests, pipelines, or embedding. It is **read-only by default**: spend and mutation are auto-denied when no human is present to approve.

```bash
git clone https://github.com/Synter-Media-AI/plugin && cd plugin/sdk
npm install
SYNTER_API_KEY=syn_... node synter-agent.mjs "/synter:report last 7 days"
```

Under the hood:

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "/synter:report last 7 days",
  options: { plugins: [{ type: "local", path: "/path/to/plugin" }] },
})) {
  // skills, agents, hooks, and the Synter MCP are all loaded
}
```

## Wire the MCP into any client

If you only want the tools (no plugin), add the Synter MCP directly.

**HTTP (recommended)**

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

**stdio (Claude Desktop)** — via [`@synterai/mcp-server`](https://www.npmjs.com/package/@synterai/mcp-server):

```json
{
  "mcpServers": {
    "synter": {
      "command": "npx",
      "args": ["@synterai/mcp-server"],
      "env": { "SYNTER_API_KEY": "YOUR_API_KEY" }
    }
  }
}
```

## Safety

Your agent can create campaigns, change budgets, and pause spend. The plugin defaults to **recommend-then-execute** and asks for explicit approval before anything spends money. It confirms the org/account before any write, uses only the real IDs the platform returns, and guards against fat-finger budgets. Reads are always free to run.

## See also

- [AI Agents](./ai-agents.md) — how Synter's agents stay transparent and controllable
- [Quick Start](../quickstart.md) — the SDK quick start
