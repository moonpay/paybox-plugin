# Installing PayBox on each agent host

PayBox is one remote MCP server, `https://api.paybox.sh/mcp`, plus a set of skills. Every host below
signs in to the same PayBox account, so the same credentials, grants, approval modes, and spend
limits apply wherever your agent runs. Revoke any host from the app's **Clients** screen.

| Host | How to install | Skills included |
|---|---|---|
| [Grok](#grok) | Plugin marketplace | Yes |
| [Claude Code](#claude-code) | Plugin marketplace | Yes |
| [Claude.ai and Claude Desktop](#claudeai-and-claude-desktop) | Custom connector | No (MCP tools only) |
| [OpenAI Codex](#openai-codex) | Plugin marketplace | Yes |
| [Any other MCP host](#any-other-mcp-host) | Remote MCP server with OAuth | Copy from `skills/` if supported |

## Grok

Add PayBox from Grok's plugin marketplace, then sign in when prompted. The manifest is
`.grok-plugin/plugin.json`.

## Claude Code

Add this repository as a marketplace and install the plugin:

```
/plugin marketplace add moonpay/paybox-plugin
/plugin install paybox@paybox
```

The plugin registers the MCP server from `.mcp.json` and loads the three skills. Run `/mcp`, select
`plugin:paybox:paybox`, and choose **Authenticate** to sign in. The manifest is
`.claude-plugin/plugin.json`, and the marketplace is `.claude-plugin/marketplace.json`.

To add only the MCP server, without the skills:

```
claude mcp add --transport http paybox https://api.paybox.sh/mcp
```

## Claude.ai and Claude Desktop

1. Open **Settings → Connectors** and choose **Add custom connector**.
2. Name it `PayBox` and enter `https://api.paybox.sh/mcp` as the URL. Leave the OAuth client fields
   empty, because PayBox registers clients automatically.
3. Choose **Connect** and sign in to PayBox.

The connector is shared by Claude.ai, Claude Desktop, and the Claude mobile apps on the same
account. The skills are not loaded on these hosts. You can upload them yourself as custom skills
from the `skills/` folder.

## OpenAI Codex

Add this repository as a marketplace, then install PayBox from the plugin browser:

```
codex plugin marketplace add moonpay/paybox-plugin
```

The manifest is `.codex-plugin/plugin.json`, and the marketplace is
`.agents/plugins/marketplace.json`. Codex asks you to sign in when the plugin is installed.

To add only the MCP server, without the skills:

```
codex mcp add paybox --url https://api.paybox.sh/mcp
codex mcp login paybox
```

## Any other MCP host

Any host that supports remote MCP servers with OAuth can use PayBox. You only need the URL.

| Setting | Value |
|---|---|
| Server URL | `https://api.paybox.sh/mcp` |
| Transport | Streamable HTTP |
| Authentication | OAuth 2.1, discovered automatically from the server's `401` response |
| Client registration | Dynamic (`/oauth/register`), so no client ID or secret is needed |
| Client type | Public client with PKCE (`S256`) |
| Scopes | `mcp`, plus `offline_access` for refresh tokens |
| Grants | Authorization code, refresh token, and device code |

The device-code grant lets hosts without a browser, such as a headless server, sign in by opening a
link on another device.

Most hosts take a JSON entry like one of these. Check your host's documentation for the file
location.

```json
{
  "mcpServers": {
    "paybox": {
      "type": "http",
      "url": "https://api.paybox.sh/mcp"
    }
  }
}
```

VS Code uses `servers` in place of `mcpServers` (in `.vscode/mcp.json`):

```json
{
  "servers": {
    "paybox": {
      "type": "http",
      "url": "https://api.paybox.sh/mcp"
    }
  }
}
```

If your host supports agent skills (`SKILL.md` folders), copy the folders under `skills/` into its
skills directory. They teach the agent the grant model, pre-spend checks, and x402 cost rules.

### If tool discovery fails after sign-in

If sign-in succeeds but the host shows no tools, the host's HTTP client may be blocked before it
reaches the server (see issue #6). Report the host name and version in an issue.

## When the signing window appears

Every money operation completes in the PayBox signing window. Hosts differ in when they draw it, and
PayBox tells each agent which case applies.

| Host | When the window appears |
|---|---|
| Grok | Only after the agent's reply is finished. Nothing is on screen while it is still typing. When the reply ends, the window appears, you sign, and the agent reports the outcome on its next turn. |
| Claude Code | To be confirmed by the PayBox team |
| Claude.ai and Claude Desktop | To be confirmed by the PayBox team |
| OpenAI Codex | To be confirmed by the PayBox team |
