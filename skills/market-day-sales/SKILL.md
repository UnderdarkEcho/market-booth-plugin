---
name: market-day-sales
description: Use to get sales for one market day or event, filtered by date and booth hours (and location if there is one). Read-only.
---
# Market-day sales
Inputs: date, hours (default the whole day, in the shop timezone), optional location.
- Shopify: `graphql_query` `orders(first:100, query:"created_at:>=START created_at:<END source_name:pos")` with `createdAt cancelledAt retailLocation{name} currentTotalPriceSet{shopMoney{amount}} totalDiscountsSet{shopMoney{amount}} transactions{gateway fees{amount{amount}}} lineItems(first:50){nodes{sku title variantTitle currentQuantity variant{id}}}`. Pass `first` (max 50) and `after` as tool arguments too; the tool defaults to 10. Put the day window in the query as UTC times (e.g. AKDT day = `T08:00:00Z` to next day `T08:00:00Z`); `get-shop-info` gives the timezone as an abbreviation like `AKDT`. The connector returns at most 50 per page. POS line SKUs can be null, so group by `variant.id` (products get renamed, so old order titles may not match current titles), falling back to title + variantTitle when the variant was deleted. `createdAt` is UTC: convert to the shop timezone before bucketing days. A gateway of `cash` only appears if cash was rung in POS. Page with `pageInfo`.
- Square: `make_api_request` service `orders` method `search` with location_ids, `date_time_filter.created_at` (RFC 3339; build the day window in the location `timezone` from `locations.list`), `state_filter.states` [COMPLETED]. Sum `total_money`; group by line item `name` + `variation_name`.
Rules: skip cancelled orders, count units net of refunds (`currentQuantity`), and split cash vs card. A shared online/store location is fine: keep orders inside the hours window, and list POS orders outside it as "unmatched", never drop them silently.
Output: orders, gross, discounts, refunds, net, cash/card, units by SKU, unmatched.
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
