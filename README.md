# SendURL MCP server

Create short links and read click analytics from your AI assistant.

[SendURL](https://sendurl.to) is a URL shortener with click analytics. Its first-party MCP server lets claude.ai,
Claude Desktop, Claude Code, ChatGPT, OpenAI Codex, Cursor, VS Code and any other
[Model Context Protocol](https://modelcontextprotocol.io) client work with your SendURL links: create them, find
and tidy them, and report on their clicks, in plain words.

- **Endpoint:** `https://api.sendurl.to/mcp`
- **Transport:** Streamable HTTP (stateless, POST only)
- **Auth:** OAuth sign-in, or an API key (`Authorization: Bearer snu_…`) with the permissions you choose
- **Price:** on every SendURL plan, Free included
- **Install:** nothing. SendURL hosts the server; this repository holds its docs, client configs and registry listing.

What it does: <https://sendurl.to/features/mcp> · Set-up guide for each client: <https://sendurl.to/guides/mcp-server> ·
Reference: <https://sendurl.to/docs#mcp>

## Contents

- [Set up](#set-up)
- [Authentication](#authentication)
- [Tools and permissions](#tools-and-permissions)
- [What the assistant sees](#what-the-assistant-sees)
- [Example prompts](#example-prompts)
- [Limits](#limits)
- [Privacy and safety](#privacy-and-safety)
- [Support](#support)
- [Licence](#licence)

## Set up

You need a SendURL account: [sign up free](https://console.sendurl.to/signup).

claude.ai, Claude Desktop, ChatGPT, Claude Code, Codex, Cursor and VS Code can all **sign in** (OAuth): you approve
the connection in your browser, and no key is stored anywhere. Clients and scripts that send a header can use an
**API key** instead. Keep the key in an environment variable or your client's secret store: **never write a key into
a config file**, least of all one inside a repository. Every example here names a variable, never the key.

### claude.ai and Claude Desktop

1. **Customize → Connectors → + Add → Add custom connector.**
2. Name: `SendURL`. URL: `https://api.sendurl.to/mcp`. If Claude shows advanced settings, keep the sign-in it
   detects and leave any request headers empty, then add the connector.
3. **Connect**, sign in to SendURL, choose the permissions and **Allow**.
4. In a chat: **+ → Connectors**, turn SendURL on.

On Team and Enterprise, an owner adds the connector in **Organization settings → Connectors** first; members then
connect from **Customize → Connectors**. Don't put an API key in the connector's request headers: Claude would keep it, and
on Team and Enterprise every member would share it.

### Claude Code

```bash
claude mcp add --transport http sendurl https://api.sendurl.to/mcp
claude mcp login sendurl
```

Your browser opens the SendURL sign-in; Claude Code stores and refreshes the token. (Or run `/mcp` inside Claude
Code.) With an API key, scopes and a shareable `.mcp.json`: [examples/claude-code.md](examples/claude-code.md).

### ChatGPT

1. Open ChatGPT's settings for apps and connectors and choose to add a custom MCP server (the menu's name depends
   on your plan and changes often).
2. Name: `SendURL`. Server URL: `https://api.sendurl.to/mcp`. Authentication: **OAuth**.
3. Confirm ChatGPT's warning about custom servers and create it, sign in to SendURL, choose the permissions and
   **Allow**. In a chat, pick SendURL from the tools menu or by typing `@`.

ChatGPT signs in; it does not send API keys. Custom MCP servers depend on your ChatGPT plan, and on a Business or
Enterprise workspace an admin may need to allow them.

### OpenAI Codex CLI

`~/.codex/config.toml` (a project's `.codex/config.toml` is read only in a project you trust):

```toml
[mcp_servers.sendurl]
url = "https://api.sendurl.to/mcp"
bearer_token_env_var = "SENDURL_API_KEY"
```

Or leave out `bearer_token_env_var` and run `codex mcp login sendurl` to sign in. See
[examples/codex.toml](examples/codex.toml).

### Cursor

`~/.cursor/mcp.json` (every project) or `.cursor/mcp.json` (one project):
[examples/cursor.json](examples/cursor.json).

```json
{
  "mcpServers": {
    "sendurl": {
      "url": "https://api.sendurl.to/mcp"
    }
  }
}
```

Cursor signs in with OAuth: your browser opens the SendURL sign-in. With an API key instead, the header reads the key
from your environment with Cursor's `${env:SENDURL_API_KEY}` syntax:
[examples/cursor-api-key.json](examples/cursor-api-key.json).

### VS Code (GitHub Copilot)

A workspace-root `.mcp.json`, or `~/.copilot/mcp-config.json` for every workspace:
[examples/vscode.json](examples/vscode.json). In this format the variable is written `${SENDURL_API_KEY}` (not
`${env:…}`, and not a bare `$SENDURL_API_KEY`, which VS Code sends as literal text); start VS Code from a shell where
it is set. To sign in instead, leave out `headers` and complete the authorization when VS Code prompts for it (or
start the sign-in from the server's actions in **MCP: List Servers**).

The older `.vscode/mcp.json` format, deprecated by VS Code and due to be removed in February 2027, can ask for the key
once and keep it as a secret: [examples/vscode-legacy.json](examples/vscode-legacy.json).

### Clients that only run local (stdio) servers

Bridge with [`mcp-remote`](https://github.com/geelen/mcp-remote). The recommended way keeps the key in a header file
only you can read, out of the process arguments and out of your client's config:

```text
# ~/.config/sendurl/headers.txt  (then: chmod 600 ~/.config/sendurl/headers.txt)
Authorization: Bearer <your key>
```

Then point the bridge at the file with `--header-file` (full path):
[examples/mcp-remote-header-file.json](examples/mcp-remote-header-file.json). Keep that file in your home folder,
never inside a repository.

Or pass the header from a variable (`export SENDURL_AUTH="Bearer $SENDURL_API_KEY"` before your client starts):
[examples/mcp-remote-env.json](examples/mcp-remote-env.json). There is no space after `Authorization:` because some
clients don't escape spaces inside `args`.

Claude Desktop itself doesn't need the bridge: use the connector above.

### Any other client

| Setting | Value |
|---|---|
| URL | `https://api.sendurl.to/mcp` |
| Transport | Streamable HTTP, stateless, POST only, no sessions |
| Sign-in | OAuth 2.1 with PKCE (S256), discovered from the 401's `WWW-Authenticate` (RFC 9728). The client identifies itself with a client ID metadata document or registers itself (dynamic client registration); the sign-in must return to an `https` or `localhost` address (custom app-link schemes are refused) |
| Or a header | `Authorization: Bearer snu_…` (an API key) |

## Authentication

**Sign in (OAuth 2.1).** Your client opens a SendURL page: sign in, check the permissions the client asks for,
untick any you don't want, approve. Only the workspace owner or an admin can approve. The connection shows under
**Connected apps** on the console's [API keys](https://console.sendurl.to/api-keys) page, where you can revoke it.
Access tokens last an hour and refresh automatically.

**API key.** Create one on the [API keys](https://console.sendurl.to/api-keys) page: **New key**, choose its reach
(all folders or one folder) and when it expires, set **Permissions** to **Custom** and tick only what the assistant
needs. Send it as `Authorization: Bearer snu_…`. To keep it out of your shell history, read it into a variable
rather than typing it in a command:

```bash
read -rs SENDURL_API_KEY && export SENDURL_API_KEY   # paste the key, then Enter
```

A console session token is refused at the MCP endpoint, and so are requests from web pages.

## Tools and permissions

17 tools, each calling the same route as the [REST API](https://sendurl.to/docs), so the same plan limits, folder
scope and error codes apply.

| Tool | What it does | Permission | Kind |
|---|---|---|---|
| `get_account` | Your workspace, plan, limits and usage | `workspace:read` | read-only |
| `list_links` | Find links by search text, folder, tag or status; newest or most-clicked first, 10 at a time | `links:read` | read-only |
| `get_link` | One link: destination, status, tags, click total and QR scans | `links:read` | read-only |
| `create_link` | Create a short link: custom slug, title, folder and tags; expiry or a password on Pro and above | `links:write` | creates |
| `update_link` | Change a link: title, tags, folder, on or off; its destination, expiry or password on Pro and above | `links:write` | **destructive** |
| `delete_link` | Delete a link for good (paid plans; Free switches links off instead) | `links:delete` | **destructive** |
| `pick_ab_test_winner` | End a link's A/B test: the winning variant becomes its destination | `links:write` | **destructive** |
| `get_link_stats` | A link's all-time clicks per day | `links:read` | read-only |
| `list_link_clicks` | A link's clicks one by one, newest first: country, device, browser and referring site | `links:read` | read-only |
| `get_analytics_overview` | Clicks for a period: totals, a daily series, the period before, top links, top tags and social sources | `analytics:read` | read-only |
| `get_analytics_breakdown` | Clicks ranked by link, country, device, browser, OS, referrer or A/B variant; by city or UTM on Pro and above | `analytics:read` | read-only |
| `list_analytics_clicks` | Clicks one by one for a period, across the workspace, a folder or one link | `analytics:read` | read-only |
| `list_folders` | Your folders, Default first (workspace keys and connections only) | `workspace:read` | read-only |
| `create_folder` | Create a folder (workspace keys and connections only) | `folders:write` | creates |
| `list_tags` | Your tags, to tag or filter links | `workspace:read` | read-only |
| `list_domains` | Your snu.to subdomains and custom domains, with their status | `workspace:read` | read-only |
| `get_domain` | One domain, with its status and DNS set-up | `workspace:read` | read-only |

`list_domains` and `get_domain` answer `domains_unavailable` until custom domains are live on SendURL.

| Permission | Console label | Allows |
|---|---|---|
| `links:read` | Read links | List and view links, their daily stats and their clicks one by one |
| `links:write` | Create and edit links | Create links, change them, pick an A/B test winner |
| `links:delete` | Delete links | Delete links (paid plans) |
| `analytics:read` | Read analytics | Overview, breakdowns and clicks across links |
| `workspace:read` | Read workspace | Account, folders, tags and domains |
| `folders:write` | Create folders | Create new folders |

Your assistant only sees the tools its permissions allow, and a folder-level key sees no folder tools. A call outside
the permissions is refused with `insufficient_scope` (an API key gets a tool error naming the `requiredScope`; a
sign-in gets a 403 the client can use to ask you for the missing permission). `update_link`, `delete_link` and
`pick_ab_test_winner` are marked destructive (`destructiveHint`), so your client can ask you to confirm before
running them; most do by default.

## What the assistant sees

Each tool returns the route's JSON, as structured content and as text. For a link's clicks, that is the time,
country, city, device, browser, operating system, source (a QR scan or not), A/B variant and the **referring site's
host only**. The assistant never gets:

- a visitor's IP address;
- the full referrer URL (its path and query can hold secrets) or the browser's user agent;
- a link's password, or any API key or token.

On Free, a click's city is withheld (`cityLocked`), as in the console.

## Example prompts

Each one works on the Free plan (7 days of analytics history) and is answered in one go.

- "Shorten https://example.com and show me its clicks."
- "Shorten https://example.com/launch and tag it Campaign."
- "How many links do I have left on my plan?"
- "Which of my links point to the old pricing page? Switch them off."
- "How many clicks has my launch link had, and how many were QR scans?"
- "Summarise this week's clicks by country and referrer."
- "What are my five most-clicked links this week?"
- "Which variant of my signup link is winning? Keep it." (keeping a winner that changes the destination is Pro and
  above)
- "Create a folder for the spring campaign and move my spring links into it."

## Limits

- Your plan's limits apply inside every tool: links, analytics history and paid features. A tool that needs a paid
  feature answers `plan_upgrade_required` with the plan that unlocks it.
- **60 tool calls a minute** per API key or connection; past it, the call is refused with a `Retry-After`.
- Analytics tools run one at a time per key or connection (a second one answers `analytics_busy`).
- **A tool call stops after 15 seconds.** Lists come 10 at a time with a cursor.
- The network rate limit of 50 requests per 10 seconds per IP address applies to the MCP endpoint too.

## Privacy and safety

- The MCP server sees only your workspace (or one folder, for a folder-level key), only with the permissions you
  gave it.
- Revoke a connection or a key on the [API keys](https://console.sendurl.to/api-keys) page; it stops working on its
  next call.
- What your assistant does with the answers is up to the assistant's provider and its terms.
- How SendURL handles your data: [Privacy Policy](https://sendurl.to/privacy). Security reports: [SECURITY.md](SECURITY.md).

## Support

- Email: [support@sendurl.to](mailto:support@sendurl.to)
- Issues with a client's set-up or this documentation: [open an issue](https://github.com/sendurl-to/sendurl-mcp/issues)
- Security: see [SECURITY.md](SECURITY.md), never a public issue

## Licence

The documentation and example configurations in this repository are under the [MIT licence](LICENSE),
copyright DRG Technologies and Consultancy. The SendURL service itself is covered by its
[Terms of Service](https://sendurl.to/terms).
