# Security

## Reporting a vulnerability

Please report security problems with the SendURL MCP server (`https://api.sendurl.to/mcp`), its OAuth sign-in or
anything else of SendURL privately, by email to **support@sendurl.to** with "Security" in the subject. Do not open
a public issue.

Include what you found, the steps to reproduce it and what an attacker could do with it. Please don't access, change
or delete data that isn't yours, and don't run tests that degrade the service for others (load tests, spam).

We will acknowledge your report, keep you updated while we fix it, and credit you if you wish once it is fixed.

## Scope

- The MCP endpoint `https://api.sendurl.to/mcp` and its OAuth endpoints on `api.sendurl.to`.
- The consent page on `console.sendurl.to`.
- The configurations in this repository.

Out of scope: the AI clients themselves (report those to their vendors), and findings that need a stolen API key or
a compromised device.

## Protecting your account

- Give an assistant only the permissions it needs: an API key with **Custom** permissions, or untick permissions
  when you approve a sign-in.
- Keep API keys in environment variables or your client's secret store, never in a file you commit or share.
- Revoke a key or a connected app on <https://console.sendurl.to/api-keys> as soon as you no longer need it or think
  it leaked; it stops working on its next call.
