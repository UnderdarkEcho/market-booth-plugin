---
name: market-day-sales
description: Use to get sales for one market day or event, filtered by date and booth hours (and location if there is one). Read-only.
---
# Market-day sales
Inputs: date, hours (default the whole day, in the shop timezone), optional location.
- Shopify: `graphql_query` `orders(first:100, query:"created_at:>=START created_at:<END source_name:pos")` with `createdAt cancelledAt retailLocation{name} currentTotalPriceSet{shopMoney{amount}} totalDiscountsSet{shopMoney{amount}} transactions{gateway fees{amount{amount}}} lineItems(first:50){nodes{sku title variantTitle currentQuantity}}`. The connector returns at most 50 per page. POS line SKUs can be null, so group by title + variantTitle. `createdAt` is UTC: convert to the shop timezone before bucketing days. A gateway of `cash` only appears if cash was rung in POS. Page with `pageInfo`.
- Square: `make_api_request` service `orders` method `search` with location_ids, `date_time_filter.created_at` (RFC 3339; build the day window in the location `timezone` from `locations.list`), `state_filter.states` [COMPLETED]. Sum `total_money`; group by line item `name` + `variation_name`.
Rules: skip cancelled orders, count units net of refunds (`currentQuantity`), and split cash vs card. A shared online/store location is fine: keep orders inside the hours window, and list POS orders outside it as "unmatched", never drop them silently.
Output: orders, gross, discounts, refunds, net, cash/card, units by SKU, unmatched.
## Connectors (read-only)
- Shopify (official connector): `graphql_query`, `list-orders`, `search_products`, `run-analytics-query`. Never call `graphql_mutation`, `set-inventory`, or `update-product`.
- Square (official MCP, https://mcp.squareup.com/mcp; tools verified in square/square-mcp-server `server.ts`): `get_service_info` and `get_type_info` first, then `make_api_request` with read methods only: `locations.list`, `orders.search`, `inventory.batchGetcounts`, `catalog.searchObjects`/`catalog.list`, `payments.list`. For local installs set `DISALLOW_WRITES=true`.
- Never invent counts. If data is missing, ask.
