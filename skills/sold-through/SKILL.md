---
name: sold-through
description: Use after a market to compare count in (packed) vs count out (left), check against POS sales, and flag recounts. Read-only.
---
# Sold-through
1. Take the packed and left counts the user gives. Log them exactly as given and ask about any that are missing.
2. Counted sold = packed − left. POS sold comes from market-day-sales.
3. Table: SKU | packed | left | counted sold | POS sold | sell-through % | flag. Flag any mismatch as "recount or unrung sale".
4. Add cash + card = day total, but only from the figures given. Note what's left that matters and the next batch to make.
5. Offer a large-print editable count sheet (item, start, sold, left, restock note) plus print copies, all in one step.
## Connectors (read-only)
- Shopify (official connector): `graphql_query`, `list-orders`, `get-inventory-levels`, `search_products`, `run-analytics-query`. Never call `graphql_mutation`, `set-inventory`, or `update-product`.
- Square (official MCP, https://mcp.squareup.com/mcp; tools verified in square/square-mcp-server `server.ts`): `get_service_info` and `get_type_info` first, then `make_api_request` with read methods only: `locations.list`, `orders.search`, `inventory.batchGetcounts`, `catalog.searchObjects`/`catalog.list`, `payments.list`. For local installs set `DISALLOW_WRITES=true`.
- Never invent counts. If data is missing, ask.
