# Claude Code

Full guide: <https://sendurl.to/guides/mcp-server#claude-code>

## Sign in (recommended)

```bash
claude mcp add --transport http sendurl https://api.sendurl.to/mcp
claude mcp login sendurl
```

Your browser opens the SendURL sign-in: approve the permissions, and Claude Code stores and refreshes the token. Or
run `/mcp` inside Claude Code and follow the steps in your browser.

## With an API key

Create a key on <https://console.sendurl.to/api-keys> with **Permissions → Custom** and only the permissions you
want. Read it into a variable, so it never appears in a command or your shell history:

```bash
read -rs SENDURL_API_KEY && export SENDURL_API_KEY   # paste the key, then Enter
```

Then add the server with a header that names the variable:

```bash
claude mcp add --transport http sendurl https://api.sendurl.to/mcp \
  --header "Authorization: Bearer $SENDURL_API_KEY"
```

Your shell fills in the key before Claude Code runs, so Claude Code saves the key itself, in your own settings. Use
this command only with the default scope or `--scope user`, **never with `--scope project`**, which would write your
key into `.mcp.json`, where the next commit shares it.

## Scope

- `--scope local` (default): you, in this project.
- `--scope user`: you, in every project.
- `--scope project`: shared with the repository through `.mcp.json`. Use it only with the sign-in command (no
  `--header`); each person then signs in.

To share a key-based connection instead, write `.mcp.json` at the root of the repository by hand, with the variable
rather than the key. Claude Code reads `${SENDURL_API_KEY}` from each person's environment when it connects, so the
file holds no key and is safe to commit:

```json
{
  "mcpServers": {
    "sendurl": {
      "type": "http",
      "url": "https://api.sendurl.to/mcp",
      "headers": { "Authorization": "Bearer ${SENDURL_API_KEY}" }
    }
  }
}
```

Each person sets `SENDURL_API_KEY` to their own key. Claude Code asks once before it uses a server from a project's
`.mcp.json`.

## Test it

Ask: "Shorten https://example.com and show me its clicks."

## Troubleshooting

- The server shows as failed although you passed a key: Claude Code doesn't fall back to a sign-in when the server
  refuses a configured `Authorization` header. Run `claude mcp list` to see the error, check the key starts with
  `snu_` and isn't revoked, and that `$SENDURL_API_KEY` was set when you ran the command; then
  `claude mcp remove sendurl` and add it again.
- The server needs sign-in again: run `claude mcp login sendurl`.
