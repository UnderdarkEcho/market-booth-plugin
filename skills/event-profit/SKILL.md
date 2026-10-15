---
name: event-profit
description: Use to work out whether a market was worth it, as net sales minus booth fee, fees and costs, and to compare events. Read-only.
---
# Event profit
- Net sales from market-day-sales. Card processing fees: Shopify `transactions{gateway fees{amount{amount}}}` where available, Square `make_api_request` service `payments` method `list` (`processing_fee`). Otherwise ask for the rate.
- Costs from event-log: booth fee, travel, other. Optional COGS = units × unit cost (Shopify `inventoryItem{unitCost}`, or ask).
- Profit = net − processing fees − booth fee − other costs − COGS. Also show profit per hour at the booth.
- Compare: rank past events by profit and profit/hour. Only report numbers you have and label any estimates.
## Connectors (read-only)
- Shopify (official connector): `graphql_query`, `list-orders`, `get-inventory-levels`, `search_products`, `run-analytics-query`. Never call `graphql_mutation`, `set-inventory`, or `update-product`.
- Square (official MCP, https://mcp.squareup.com/mcp; tools verified in square/square-mcp-server `server.ts`): `get_service_info` and `get_type_info` first, then `make_api_request` with read methods only: `locations.list`, `orders.search`, `inventory.batchGetcounts`, `catalog.searchObjects`/`catalog.list`, `payments.list`. For local installs set `DISALLOW_WRITES=true`.
- Never invent counts. If data is missing, ask.
