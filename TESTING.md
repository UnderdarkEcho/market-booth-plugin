# Testing results (2026-10-14, read-only, Shopify connector `graphql_query` only, no mutations)

| Skill | Test | Result |
|---|---|---|
| market-day-sales | POS orders for 2026-09-19 (local), using the orders query in the skill | PASS: 23 orders, net $975.00, card fees $27.67, 61 units, first/last sale 11:57/17:17. Top: Aches & Pains 13, Spruce Sentry 8, Tea 8. |
| market-day-sales | 2026-09-12 | PASS: 26 orders, net $730.00, fees $21.60, 65 units. |
| pack-list | last 2 market days, +25% buffer, vs stock from the productVariants query | PASS: Aches & Pains avg 10.5, pack 14, on hand 69 OK. Spruce Sentry avg 7.5, pack 10, on hand 51 OK. Tea avg 4, pack 5, **on hand 0, flagged**. |
| sold-through | sample: packed Aches 20 / left 7 vs POS 13 | PASS: counted 13 = POS 13, 65%. A sample with left 5 → counted 15 vs POS 13, flagged for recount. |
| restock | Tea, 2 upcoming events, batch 12 | PASS: need 4×2+5−0 = 13, so 2 batches. |
| event-profit | 2026-09-19 with a sample $40 booth fee and COGS from unitCost where set | PASS: 975 − 27.67 − 40 = $907.33 before COGS. COGS is partial, because some variants have no unitCost, so it's labelled an estimate. |
| event-log | CSV columns | PASS (spec review). |
| Square | No Square account connected | NOT RUN. Tool and method names verified from the square/square-mcp-server source. |

## Fixes made
- Square: the real methods are `orders.search`, `inventory.batchGetcounts` and `payments.list` through `make_api_request` (not `SearchOrders`/`BatchRetrieveInventoryCounts`/`ListPayments`).
- Shopify: replaced `get-inventory-levels` with a validated `productVariants` query. Fees field is `transactions{fees{amount{amount}}}`. Money fields need `{shopMoney{amount}}`. Max 50 per page.
- POS line items can have a null `sku`, so group by title + variantTitle.
- `createdAt` is UTC, so convert to the shop timezone before bucketing days.
- All POS payments were `shopify_payments` (card). Cash shows only if it's rung in POS.
Note: the connector validates every query against Shopify's pinned schema before running it, and all the queries above passed. `validate_graphql_codeblocks` is a no-op.
