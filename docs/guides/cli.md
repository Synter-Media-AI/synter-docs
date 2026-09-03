# Synter CLI

```bash
npm install -g synter
```

The CLI and the SDKs are the same product from two angles. Both authenticate
with a `syn_` API key and both dispatch to
`POST https://syntermedia.ai/api/v1/tools/run`. Use the CLI when the caller is a
shell or an agent with shell access; use an SDK when the caller is your
application.

## Authentication

Three headers are accepted and they are equivalent. Use whichever your client
makes easiest:

```
Authorization: Bearer syn_...
X-Synter-Key: syn_...
X-API-Key: syn_...
```

Keys start with `syn_`. A key sent under any other header, or with a different
prefix, is treated as absent.

For the CLI, `synter login` stores the key for you:

```bash
synter login --no-browser
```

This prints a URL and a short code instead of opening a browser, which is what
you want on a server or inside an agent. A human opens the URL once and approves
the device. The CLI polls until it is approved, then stores the key.

`synter signup --no-browser` is the same handshake for an account that does not
exist yet.

## Discovering what you can run

```bash
synter tools --json --all        # requires 0.7.0
```

Each entry carries `name`, `kind` (`readonly` or `destructive`),
`required_scope`, `credit_cost`, and `executable`.

**Read `executable` before you call anything.** The tool catalog is the MCP
surface, which is larger than what the run endpoint dispatches. A tool with
`executable: false` is reachable over MCP but has no script behind that name, so
`synter execute` will refuse it.

The response also reports `total`, `page` and `has_more`. Page through with
`--page`, or pass `--all` for one listing. If `source` is `static_manifest`
rather than `catalog`, the live catalog was unreachable and you are looking at a
small bundled fallback, not the full surface.

## Running a tool

```bash
synter execute pull_google_ads --arg date_range=LAST_30_DAYS --json
```

`--arg key=value` is repeatable. A bare `--arg key` sends a boolean flag.
`--platform` selects which connected account's credentials to use.

Names are exact. `pull_google_ads` is a script; `pull_google_ads_performance` is
the MCP tool name for the same capability and is **not** dispatchable here. When
a name is wrong the error names it and lists what is available:

```json
{
  "success": false,
  "error": "UNKNOWN_SCRIPT",
  "message": "Script 'pull_google_ads_performance' not in whitelist. Available: [...]",
  "http_status": 400,
  "script_name": "pull_google_ads_performance"
}
```

Other codes worth handling: `SCOPE_MISSING` (mint a key with the scope named in
the body), `INSUFFICIENT_CREDITS`, and `PAYMENT_METHOD_REQUIRED`.

## Ad account access

```bash
synter connect google --url --json
```

This mints a single-use OAuth URL. The agent cannot complete the grant — the ad
platforms require a human to approve access in their own UI. Hand the URL over,
then poll `synter execute list_connected_accounts` until the account appears.
The link is single-use and short-lived, so mint a fresh one if it expires.

## Billing

```bash
synter billing status --json
synter billing checkout --plan solo --json
```

`checkout` returns a URL for a human to enter a card. Add `--wait` to make it a
single step: the CLI prints the link, opens it, and blocks until the purchase
lands, then prints the new tier.

```bash
synter billing checkout --plan solo --wait
```

The link is printed before anything blocks, so it is still usable if no browser
opens. `--no-browser` prints without launching one, which is what you want on a
server. `--timeout <seconds>` bounds the wait, default 900. A timeout is
reported as pending rather than failure, with the link repeated, because the
checkout page usually still works.

Under `--json` the progress lines go to stderr and stdout carries exactly one
document, so `synter billing checkout --plan solo --wait --json | jq` is safe.

Once a card is saved, a plan can be started without a browser at all:

```bash
synter billing subscribe --plan solo --confirm --json   # requires 0.7.0
```

Both `--plan` and `--confirm` are required. Without `--confirm` the command
reports what it would buy and exits non-zero without calling the API, so you can
always dry-run it. This is the only CLI command that charges a card.

It refuses with `402` and a checkout URL when no card is saved, and with `409`
when the workspace is already on a paid plan — plan changes go through the
audited change-plan flow rather than this endpoint.

## API keys

```bash
synter keys list --json
synter keys create --scopes reporting:read,campaigns:read --json
synter keys revoke <id>
```

A key can only mint children whose scopes are a subset of its own. The key
material is printed once, at creation.

## Exit codes and output

Every command listed above exits non-zero on failure and, with `--json`, prints
exactly one JSON document on stdout. Progress and prompts go to stderr, so
`command --json | jq` is safe.

## Commands that are not agent-safe

`chat`, `pull`, `analyze`, `create`, `pause`, `enable` and `budget` are
conveniences for a person at a terminal. They send an English instruction to the
assistant and several of them block on interactive confirmation prompts, so they
hang or fail when there is no TTY. Their output is prose, not a data contract.

For automation use `synter execute` with an explicit script name, which is
deterministic and has no model in the loop.

## What still needs a human

Three things, once each, and none of them is a Synter limitation:

1. **Sign in** — approving the device code.
2. **Enter a card** — Stripe collects it in the browser.
3. **Grant ad-platform access** — each platform requires its own OAuth consent.

Everything after that is reachable from the CLI, the SDKs and MCP.
