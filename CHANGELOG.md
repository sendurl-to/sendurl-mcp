# Changelog

The server is hosted by SendURL; this log records changes to its tools, auth and limits as clients see them.

## 1.0.0

- First public release of the SendURL MCP server at `https://api.sendurl.to/mcp` (Streamable HTTP, stateless).
- Sign-in with OAuth 2.1, or an API key with chosen permissions.
- 17 tools: links (list, get, create, update, delete, A/B test winner, daily stats, clicks), analytics (overview,
  breakdown, clicks), folders, tags and domains. `update_link`, `delete_link` and `pick_ab_test_winner` are marked
  destructive.
- Limits: 60 tool calls a minute per key or connection, analytics tools one at a time, 15 s per tool call.
