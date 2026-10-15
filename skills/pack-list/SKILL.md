---
name: pack-list
description: Use before a market, fair, or pop-up to decide what to bring. Builds a pack list from sales at recent events plus a buffer. Read-only.
---
# Pack list
1. Get the upcoming event and the last N similar events (default 3) from event-log.
2. For each past event, get units sold per SKU (see market-day-sales).
3. Suggested pack = ceil(avg sold × (1 + buffer)), with a default buffer of 25% and a minimum of 1 for SKUs that have sold before.
4. Show on-hand stock: Shopify `graphql_query` `productVariants(first:50, query:"product_status:active"){pageInfo{hasNextPage endCursor} nodes{id title sku product{title} inventoryQuantity inventoryItem{tracked unitCost{amount}}}}`; Square `make_api_request` service `inventory` method `batchGetcounts` (keep `state` IN_STOCK). Join sales to stock on variant id. Items sold but missing from active variants (archived, renamed or one-off items) show on hand as "no record" and ask.
5. Output a large-print table to tick off: item | on hand | pack | basis. Flag SKUs where pack > on hand.
If there's no history, say so and ask for counts. Don't guess.
## Connectors (read-only)
- Shopify (official connector): `graphql_query`, `list-orders`, `search_products`, `run-analytics-query`. Never call `graphql_mutation`, `set-inventory`, or `update-product`.
- Square (official MCP, https://mcp.squareup.com/mcp; tools verified in square/square-mcp-server `server.ts`): `get_service_info` and `get_type_info` first, then `make_api_request` with read methods only: `locations.list`, `orders.search`, `inventory.batchGetcounts`, `catalog.searchObjects`/`catalog.list`, `payments.list`. For local installs set `DISALLOW_WRITES=true`.
- Never invent counts. If data is missing, ask.

## If Square won't connect
If a Square tool call fails with an auth error (`401`, `UNAUTHORIZED`, "not connected", "authentication required"), or the user says Square sign-in showed **"Invalid redirect URI"**, the OAuth connection (the primary path) isn't working. Don't retry over and over. Instead:
1. Have the user try **Authenticate** on the Square connector once more. If it says "Invalid redirect URI", tell them it's an allowlist setup issue, not something they did.
2. If OAuth still fails, point them to the secondary fallback in `skills/SQUARE-CONNECT.md` → "Secondary fallback: your own Square access token" (Dashboard → app → Credentials → Sandbox or Production token, stored as a secret, local server with `DISALLOW_WRITES=true`).
3. **Never ask the user to paste a token into chat.** If they paste one, don't repeat it or store it. Tell them to revoke it in the Dashboard and save a new one as a secret.
4. Meanwhile, carry on with Shopify data if it's connected, or with numbers the user gives you, and label the Square part "not connected".
