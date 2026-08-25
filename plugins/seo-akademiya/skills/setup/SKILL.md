---
name: setup
description: Configure SEO Akademiya MCP access. Use when the seo-akademiya MCP server is not connected, the SEO_AKADEMIYA_TOKEN environment variable is missing, seo-akademiya tools return 401/Unauthorized, or the user asks to set up, connect, or configure SEO Akademiya access or mentions their access token.
---

# SEO Akademiya access setup

You help the user store their personal SEO Akademiya access token so the `seo-akademiya` MCP server can connect.

## Steps

1. Check whether the token is already configured: read `~/.claude/settings.json` and look for `env.SEO_AKADEMIYA_TOKEN`.
2. If it is present and the user reports errors, the token may be revoked — tell the user to request a new token from the administrator.
3. If it is missing, ask the user (in their language) to paste their personal access token. It looks like `seoak_` followed by 48 hex characters. The token is issued personally by the administrator.
4. Validate the format (`seoak_[0-9a-f]{48}`). If it does not match, ask again.
5. Update `~/.claude/settings.json`: merge `{"env": {"SEO_AKADEMIYA_TOKEN": "<token>"}}` into the existing JSON without removing any existing keys. Create the file with exactly that content if it does not exist.
6. Tell the user to fully quit and restart Claude Code, then verify the connection by typing `/mcp` — the `seo-akademiya` server should show as connected.
7. Recommend the user clear this conversation (`/clear`) afterwards, since the pasted token remains in the chat history.

## Notes

- Never send the token anywhere except writing it into `~/.claude/settings.json`.
- Never commit the token to any repository.
- If the server still fails after restart, the token may be revoked or the account disabled — the user should contact the administrator.
