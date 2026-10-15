---
name: restock
description: Use to plan what to make or reorder before upcoming events, rounded to the maker's batch sizes. Read-only.
---
# Restock
1. Upcoming events from event-log, and the sales rate per event from the last N events (market-day-sales).
2. Need = avg sold per event × upcoming events + pack buffer − on hand (inventory as in pack-list).
3. Round up to the user's batch size per product (ask once and remember, e.g. "soap: 12 bars/batch").
4. Output: SKU | on hand | need | batches to make | reason. Suggestions only. Never change inventory.
## Connectors (read-only)
- Shopify (official connector): `graphql_query`, `list-orders`, `search_products`, `run-analytics-query`. Never call `graphql_mutation`, `set-inventory`, or `update-product`.
- Square (official MCP, https://mcp.squareup.com/mcp; tools verified in square/square-mcp-server `server.ts`): `get_service_info` and `get_type_info` first, then `make_api_request` with read methods only: `locations.list`, `orders.search`, `inventory.batchGetcounts`, `catalog.searchObjects`/`catalog.list`, `payments.list`. For local installs set `DISALLOW_WRITES=true`.
- Never invent counts. If data is missing, ask.

## If Square won't connect
If a Square tool call fails with an auth error (`401`, `UNAUTHORIZED`, "not connected", "authentication required"), or the user says Square sign-in showed **"Invalid redirect URI"**, the hosted OAuth connection isn't working. Don't retry over and over. Instead:
1. Tell the user that Square's hosted sign-in doesn't work in this app yet. It's a known Square allowlist issue, not something they did.
2. Point them to `skills/SQUARE-CONNECT.md` → "Fallback: your own Square access token" (Dashboard → app → Credentials → Sandbox or Production token, stored as a secret, local server with `DISALLOW_WRITES=true`).
3. **Never ask the user to paste a token into chat.** If they paste one, don't repeat it or store it. Tell them to revoke it in the Dashboard and save a new one as a secret.
4. Meanwhile, carry on with Shopify data if it's connected, or with numbers the user gives you, and label the Square part "not connected".
