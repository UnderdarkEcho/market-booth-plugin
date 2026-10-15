---
name: event-profit
description: Use to work out whether a market was worth it, as net sales minus booth fee, fees and costs, and to compare events. Read-only.
---
# Event profit
- Net sales from market-day-sales. Card processing fees: Shopify `transactions{gateway fees{amount{amount}}}` where available, Square `make_api_request` service `payments` method `list` (`processing_fee`; it can be missing or arrive late, so if it is absent ask for the rate and label fees as estimated).
- Costs from event-log: booth fee, travel, other. Optional COGS = units × unit cost (Shopify `inventoryItem{unitCost}`, or ask).
- Profit = net − processing fees − booth fee − other costs − COGS. Also show profit per hour at the booth, using booth hours from event-log (ask if missing; first/last sale times are not booth hours).
- Compare: rank past events by profit and profit/hour. Only report numbers you have and label any estimates.
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
