# Changelog

## 0.2.0 (2026-10-15)
- **Square personal-token fallback.** New `skills/SQUARE-CONNECT.md` covers creating a Sandbox or Production token in the Square Developer Dashboard, storing it as a secret (never in chat or the repo), and running the official `square-mcp-server` locally with `DISALLOW_WRITES=true`.
- **Failed-OAuth detection.** Every skill now spots Square auth failures (401, UNAUTHORIZED, "Invalid redirect URI") and points to the token fallback instead of retrying. Skills never ask for a token in chat.
- README: new "Connecting Square" section describing the known hosted-OAuth redirect issue.
- `notes/square-redirect-request.md`: draft request to Square developer support to allowlist the app's OAuth redirect URI (not sent).
- Version bumped to 0.2.0.

## 0.1.0 (2026-10-14)
- First release: pack-list, market-day-sales, sold-through, restock, event-profit, event-log on the official Shopify connector and Square MCP (read-only).
