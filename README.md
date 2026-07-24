# Synter Documentation

**The developer-first platform for multi-platform ad management.**

Ship ads like you ship code. One API for Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, X, Amazon, and more — 20+ ad platforms.

> **Full documentation lives at [docs.syntermedia.ai](https://docs.syntermedia.ai)** — quickstart, playground, API reference, and MCP server docs. This repo hosts the SDK guides and reference material.

## 📚 Documentation

- [Quick Start Guide](./docs/quickstart.md) - Get started in 5 minutes
- [TypeScript SDK](./sdk/typescript/README.md) - API reference for Node.js/TypeScript
- [Python SDK](./sdk/python/README.md) - API reference for Python
- [Guides](./docs/guides/README.md) - UTM management, conversions, agents
- [Claude Plugin](./docs/guides/claude-plugin.md) - Run Synter inside Claude Code / Claude Desktop

## 🚀 Quick Install

### TypeScript/Node.js

```bash
npm install @synterai/sdk-js
```

```typescript
import { Synter } from '@synterai/sdk-js';

const synter = new Synter(process.env.SYNTER_API_KEY);

// List campaigns
const campaigns = await synter.campaigns.list({ platform: 'google' });

// Pull performance
const performance = await synter.analytics.getPerformance({
  platform: 'google',
  date_range: 'LAST_7_DAYS'
});
```

### Python

```bash
pip install synter
```

```python
from synter import Synter

client = Synter(api_key="syn_...")

# List campaigns
campaigns = client.campaigns.list(platform="google")

# Pull performance
performance = client.analytics.get_performance(platform="google")
```

### CLI

```bash
npm install -g synter
synter login   # device-code browser login — no copy-paste needed
synter         # start the interactive session
```

### MCP Server (for AI agents)

Connect Claude, Cursor, or any MCP client directly to your ad accounts:

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

Or via stdio with [`@synterai/mcp-server`](https://www.npmjs.com/package/@synterai/mcp-server). No account yet? The MCP server can onboard you: call `synter_onboarding_start(email)`, click the magic link in your inbox, then poll `synter_onboarding_status(session_token)`.

## ✨ Features

- **Multi-platform**: Google, Meta, LinkedIn, Microsoft, Reddit, TikTok, X, Amazon, Pinterest, Snap, Spotify, and more
- **UTM Management**: Auto-generated, platform-specific UTM parameters
- **Conversion Tracking**: Unified tracking across all platforms
- **AI Agents**: Transparent, controllable optimization with dry-run-by-default safety
- **Free GA4 tools**: Google Analytics 4 reporting costs no credits
- **Bring Your Own AI**: Export raw data for custom ML models

## 🔗 Links

- [Documentation](https://docs.syntermedia.ai)
- [Website](https://syntermedia.ai)
- [Developer Portal / API Keys](https://syntermedia.ai/developer)
- [Discord Community](https://discord.gg/syntermedia)

## 📄 License

MIT
