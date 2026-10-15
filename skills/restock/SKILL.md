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
