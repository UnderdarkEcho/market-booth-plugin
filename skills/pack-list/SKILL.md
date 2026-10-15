---
name: pack-list
description: Use before a market, fair, or pop-up to decide what to bring. Builds a pack list from sales at recent events plus a buffer. Read-only.
---
# Pack list
1. Get the upcoming event and the last N similar events (default 3) from event-log.
2. For each past event, get units sold per SKU (see market-day-sales).
3. Suggested pack = ceil(avg sold × (1 + buffer)), with a default buffer of 25% and a minimum of 1 for SKUs that have sold before.
4. Show on-hand stock: Shopify `graphql_query` `productVariants(first:50, query:"product_status:active"){nodes{title sku product{title} inventoryQuantity inventoryItem{tracked unitCost{amount}}}}`; Square `make_api_request` inventory `BatchRetrieveInventoryCounts`.
5. Output a large-print table to tick off: item | on hand | pack | basis. Flag SKUs where pack > on hand.
If there's no history, say so and ask for counts. Don't guess.
## Connectors (read-only)
- Shopify (official connector): `graphql_query`, `list-orders`, `get-inventory-levels`, `search_products`, `run-analytics-query`. Never call `graphql_mutation`, `set-inventory`, or `update-product`.
- Square (official MCP, https://mcp.squareup.com/mcp; tools verified in square/square-mcp-server `server.ts`): `get_service_info` and `get_type_info` first, then `make_api_request` with read methods only: `locations.list`, `orders.search`, `inventory.batchGetcounts`, `catalog.searchObjects`/`catalog.list`, `payments.list`. For local installs set `DISALLOW_WRITES=true`.
- Never invent counts. If data is missing, ask.
