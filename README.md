<p align="center"><img src="assets/logo.png" width="96" alt="ZRouter logo"></p>

# ZRouter MCP Server

Manage your [ZRouter](https://zrouter.si) account from any AI assistant. Check spend, search request logs, create virtual model routers, issue API keys, and set budgets and rate limits in plain language.

```text
https://zrouter.si/mcp
```

ZRouter is the Super Intelligence Router: one OpenAI-compatible API key for chat, images, audio, and video, routed to the right model and provider. This MCP server is the management side of your account. It runs as a hosted remote server, so there is nothing to install.

- **Transport:** Streamable HTTP
- **Sign-in:** OAuth 2.1 with PKCE and dynamic client registration, or a bearer token
- **Scope:** your own account only, with the same rules as the dashboard
- **Tools:** 24 (14 read-only, 4 write, 6 marked destructive)

## Contents

- [Quick start](#quick-start)
- [Connect a client](#connect-a-client)
- [Use a token instead of sign-in](#use-a-token-instead-of-sign-in)
- [What you can ask](#what-you-can-ask)
- [Tools](#tools)
- [Security](#security)
- [Authentication details](#authentication-details)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Links](#links)

## Quick start

1. Create a free account at [zrouter.si](https://zrouter.si/login?mode=signup).
2. Add `https://zrouter.si/mcp` as a remote MCP server in your client.
3. Approve access on the ZRouter page that opens.
4. Ask: "How much did I spend this week, by model?"

Clients that support MCP sign-in only need the URL. There is no token to copy or renew.

## Connect a client

### Claude (claude.ai and Claude Desktop)

Settings → Connectors → **Add custom connector**. Paste `https://zrouter.si/mcp`, then select **Connect** and approve access.

### ChatGPT

Settings → Apps & Connectors → create a connector with the URL `https://zrouter.si/mcp` and **OAuth** authentication. Custom connectors may require developer mode, depending on your plan.

### Claude Code

```bash
claude mcp add --transport http zrouter https://zrouter.si/mcp
```

Then run `/mcp` inside Claude Code and choose **Authenticate**.

### Cursor

This repository is also a Cursor plugin (`.cursor-plugin/plugin.json`). Or add the server to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "zrouter": {
      "url": "https://zrouter.si/mcp"
    }
  }
}
```

### VS Code

Add the server to `.vscode/mcp.json` or your user configuration:

```json
{
  "servers": {
    "zrouter": {
      "type": "http",
      "url": "https://zrouter.si/mcp"
    }
  }
}
```

### Other clients

Add a remote MCP server with the URL `https://zrouter.si/mcp` and the Streamable HTTP transport. Choose OAuth sign-in when the client asks.

Client configuration formats change between versions. If a snippet above does not match your client, follow its own MCP documentation and use the same URL.

## Use a token instead of sign-in

For scripts and clients without MCP sign-in, use an MCP token.

1. Open **CLI and MCP access** in your [ZRouter dashboard](https://zrouter.si/dashboard).
2. Name the token after your client and select **Create MCP token**.
3. Copy it right away. It is shown once and expires after 30 days.

Send it as a bearer token:

```bash
claude mcp add --transport http zrouter https://zrouter.si/mcp \
  --header "Authorization: Bearer YOUR_MCP_TOKEN"
```

```json
{
  "mcpServers": {
    "zrouter": {
      "url": "https://zrouter.si/mcp",
      "headers": { "Authorization": "Bearer YOUR_MCP_TOKEN" }
    }
  }
}
```

Use an MCP token, not an account API key. API keys only send model requests and cannot manage your account, so a leaked API key cannot create keys or change budgets. Keep MCP tokens out of shared repository configuration, and revoke tokens you no longer use from the same dashboard page.

## What you can ask

- "How much did I spend this week, by model?"
- "Show the requests that failed with 429 today."
- "Why did my last request to the support bot fail?"
- "Create a router called support-bot that splits traffic between two models."
- "Route easy prompts to a cheap model and hard ones to my strongest model."
- "Create an API key for staging and cap my spend at $10 a day."
- "Limit the staging user path to 60 requests a minute."

Connecting this server does not change your assistant's own model provider. To send model requests through ZRouter, set up an API key with one of the [integration guides](https://zrouter.si/docs/integrations).

## Tools

Every tool runs inside the authenticated account. Read-only tools are annotated `readOnlyHint`. Deletes and resets are annotated `destructiveHint`, so clients that confirm risky actions ask you first.

### Account

| Tool | What it does | Type |
| --- | --- | --- |
| `account_info` | The signed-in account and its credit balance | Read-only |
| `credit_ledger` | Credit purchases and charges | Read-only |

### Usage

| Tool | What it does | Type |
| --- | --- | --- |
| `usage_summary` | Total requests, tokens, and cost for a time window | Read-only |
| `usage_daily` | Usage and cost over time, by day, week, or month | Read-only |
| `usage_by_model` | Usage and cost grouped by model | Read-only |
| `usage_log` | Per-request entries with tokens and cost | Read-only |

Usage tools accept `days` (default 30), `start_date` and `end_date` (`YYYY-MM-DD`), `user_path`, `model`, and `provider`.

### Logs

| Tool | What it does | Type |
| --- | --- | --- |
| `logs_search` | Search request logs by status code, error type, or text | Read-only |
| `logs_get` | Full detail of one log entry (`log_id` from `logs_search`) | Read-only |
| `logs_stats` | Request counts, error rates, and latency | Read-only |

### Models and routers

| Tool | What it does | Type |
| --- | --- | --- |
| `models_list` | Models this account can use | Read-only |
| `virtual_models_list` | Your routers, the routing strategies, and your model name prefix | Read-only |
| `virtual_models_upsert` | Create or update a router: up to 32 targets, weights, and a strategy such as `round_robin` or `super_intelligence` | Write |
| `virtual_models_delete` | Delete a router | Destructive |

A router is a stable model name your app calls. The `super_intelligence` strategy scores each request by difficulty and sends it to `simple_targets`, `standard_targets`, or `complex_targets`.

### API keys

| Tool | What it does | Type |
| --- | --- | --- |
| `api_keys_list` | List keys. Secrets are never returned. | Read-only |
| `api_keys_create` | Create a key. The secret is returned once. | Write |
| `api_keys_delete` | Delete a key. Requests using it stop working immediately. | Destructive |

### Budgets

| Tool | What it does | Type |
| --- | --- | --- |
| `budgets_list` | Budgets with current spend | Read-only |
| `budgets_upsert` | Create or update a USD spend limit (hourly, daily, weekly, or monthly) | Write |
| `budgets_delete` | Delete a budget | Destructive |
| `budgets_reset` | Reset one budget's current spend to zero | Destructive |

### Rate limits

| Tool | What it does | Type |
| --- | --- | --- |
| `rate_limits_list` | Rate limits with current counters | Read-only |
| `rate_limits_upsert` | Create or update a request or token limit per minute, hour, or day | Write |
| `rate_limits_delete` | Delete a rate limit | Destructive |
| `rate_limits_reset` | Reset one rate limit's counters | Destructive |

Budgets apply to a `user_path` or `label`. Rate limits apply to a `user_path`, `provider`, or `model`. Set `per_child` to apply a rule separately to each child user path.

## Security

- **Your account only.** Each tool call goes through the same customer API as the dashboard, with your credential, so authentication, account scoping, and validation are identical.
- **Separate credentials.** MCP access uses OAuth tokens or MCP tokens. Account API keys cannot manage the account.
- **Revocable.** Disconnect apps under **CLI and MCP access → Connected apps**, and revoke MCP tokens on the same page.
- **Bounded results.** A single tool result is capped at 256 KB so a large log page cannot flood your assistant's context.
- **Secrets in history.** A new API key's secret appears in the `api_keys_create` result, and so in your assistant's conversation history. Treat that history accordingly, or create keys in the dashboard.

## Authentication details

For client developers. The server follows the MCP authorization specification.

| | |
| --- | --- |
| MCP endpoint | `https://zrouter.si/mcp` (POST, Streamable HTTP, stateless, JSON responses) |
| Protected resource metadata | `https://zrouter.si/.well-known/oauth-protected-resource/mcp` (RFC 9728) |
| Authorization server metadata | `https://zrouter.si/.well-known/oauth-authorization-server` (RFC 8414) |
| Authorization endpoint | `https://zrouter.si/oauth/authorize` |
| Token endpoint | `https://zrouter.si/oauth/token` |
| Dynamic client registration | `https://zrouter.si/oauth/register` (RFC 7591) |
| Grant types | `authorization_code`, `refresh_token` |
| PKCE | Required, `S256` |
| Client authentication | `none` (public clients) |
| Scope | `account` |

A request without a valid token gets `401` with:

```text
WWW-Authenticate: Bearer resource_metadata="https://zrouter.si/.well-known/oauth-protected-resource/mcp", scope="account"
```

Quick check with an MCP token:

```bash
curl -s https://zrouter.si/mcp \
  -H "Authorization: Bearer YOUR_MCP_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Troubleshooting

| Problem | Fix |
| --- | --- |
| `401` with `invalid_token` | The token expired (MCP tokens last 30 days) or was revoked, or an account API key was sent instead of an MCP token. Sign in again or create a new MCP token. |
| The client cannot sign in | Make sure it supports remote MCP with OAuth. Otherwise use an MCP token. |
| A tool reports a validation error | The tool returns the same message as the dashboard. Fix the argument it names and retry. |
| The assistant uses the wrong account | Disconnect the app under **Connected apps** and sign in again with the right account. |

## FAQ

**Is there anything to install or host?**
No. ZRouter hosts the server at `https://zrouter.si/mcp`.

**Does it cost anything?**
No. Account management through MCP is free. Model requests you send with your API keys draw from your prepaid credit as usual.

**Can the assistant reach another account?**
No. Every tool runs inside the account that signed in.

**Does connecting MCP change which model my assistant uses?**
No. It only adds account management tools.

**Can I manage the account from a terminal instead?**
Yes, with the [zctl CLI](https://zrouter.si/cli).

## Links

- Website: <https://zrouter.si>
- MCP guide: <https://zrouter.si/docs/mcp>
- API guide: <https://zrouter.si/docs/api>
- Integrations: <https://zrouter.si/docs/integrations>
- zctl CLI: <https://zrouter.si/cli>
- Status: <https://zrouter.si/status>
- Smithery https://smithery.ai/badge/sali/zrouter
[![smithery badge](https://smithery.ai/badge/sali/zrouter)](https://smithery.ai/servers/sali/zrouter)

## License

The contents of this directory are available under the [MIT License](LICENSE).
