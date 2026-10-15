> All product names, prices, counts and dates in this file are **dummy example data**.

# Testing results (2026-10-14, read-only, Shopify connector `graphql_query` only, no mutations)

| Skill | Test | Result |
|---|---|---|
| market-day-sales | POS orders for 2025-01-11 (local), using the orders query in the skill | PASS: 12 orders, net $480.00, card fees $14.00, 30 units, first/last sale 10:05/14:50. Top: Sample Salve A 6, Sample Soap B 4, Sample Tea C 3. |
| market-day-sales | 2025-01-04 | PASS: 10 orders, net $400.00, fees $11.60, 25 units. |
| pack-list | last 2 market days, +25% buffer, vs stock from the productVariants query | PASS: Sample Salve A avg 10.5, pack 14, on hand 40 OK. Sample Soap B avg 7.5, pack 10, on hand 30 OK. Sample Tea C avg 4, pack 5, **on hand 0, flagged**. |
| sold-through | sample: packed Salve A 20 / left 7 vs POS 13 | PASS: counted 13 = POS 13, 65%. A sample with left 5 → counted 15 vs POS 13, flagged for recount. |
| restock | Sample Tea C, 2 upcoming events, batch 12 | PASS: need 4×2+5−0 = 13, so 2 batches. |
| event-profit | 2025-01-11 with a sample $40 booth fee and COGS from unitCost where set | PASS: 480 − 14.00 − 40 = $426.00 before COGS. COGS is partial, because some variants have no unitCost, so it's labelled an estimate. |
| event-log | CSV columns | PASS (spec review). |
| Square | No Square account connected | NOT RUN. Tool and method names verified from the square/square-mcp-server source. |

## Fixes made
- Square: the real methods are `orders.search`, `inventory.batchGetcounts` and `payments.list` through `make_api_request` (not `SearchOrders`/`BatchRetrieveInventoryCounts`/`ListPayments`).
- Shopify: replaced `get-inventory-levels` with a validated `productVariants` query. Fees field is `transactions{fees{amount{amount}}}`. Money fields need `{shopMoney{amount}}`. Max 50 per page.
- POS line items can have a null `sku`, so group by title + variantTitle.
- `createdAt` is UTC, so convert to the shop timezone before bucketing days.
- All POS payments were `shopify_payments` (card). Cash shows only if it's rung in POS.
Note: the connector validates every query against Shopify's pinned schema before running it, and all the queries above passed. `validate_graphql_codeblocks` is a no-op.

# Square sandbox results (2026-10-14, connect.squareupsandbox.com, REST equivalents of the MCP methods)

Seeded: 4 items / 5 variations (Sample Soap B 1 oz $12 & 2 oz $20, Sample Salve A $25, Sample Soap D $9, Sample Tea C $10), counts 20/10/15/12/6, 5 orders paid with `cnon:card-nonce-ok`.

| Skill | Result |
|---|---|
| locations.list | PASS: 1 location (Sample Test Location, timezone UTC) |
| market-day-sales | PASS: orders.search COMPLETED → 5 orders, $223.00, 14 units, correct per-variation counts |
| pack-list | PASS: inventory.batchGetcounts returned on-hand 17/8/11/9/4 |
| sold-through | PASS: packed − sold = on-hand for all 5 variations (15–33%) |
| restock | PASS: no restock flagged at this stock level (math verified) |
| event-profit | PASS with caveat: payments.list returned 5 payments but no `processing_fee` in sandbox → $223 − $0 − $40 booth = $183 (fees must be estimated) |
| catalog.searchObjects | PASS: 4 items |
| event-log | PASS (spec, no Square calls) |

Fixes: pack-list still named `BatchRetrieveInventoryCounts` → `batchGetcounts`; removed `get-inventory-levels` from tool lists; market-day-sales now specifies `state_filter` and location-timezone day window; event-profit handles missing `processing_fee`.

# In-app test (Shopify connector)
2026-10-14, Grok Bot app, a connected Shopify dev store (all names and figures below are dummy examples) via the official Shopify connector (`get-shop-info`, `graphql_query` only; no writes). Shop timezone (example) AKST.

| Skill | Result |
|---|---|
| market-day-sales 2025-01-11 | PASS: 12 POS orders, 0 cancelled, net $480.00, discounts $10.00, refunds $0, fees $14.00, all `shopify_payments` (no cash rung), 1 location, sales 10:05–14:50 AKDT. Top: Sample Salve A 13, Sample Soap B 8, Sample Tea C 8. |
| market-day-sales, most recent | Only POS order after 1/11 is one $60 order on 2025-01-15 (evening, not a market). Most recent market days before that: 1/4 (10 orders, $400; earlier table's 26/$730 did not use the full local day) and 12/28 (8, $310). |
| pack-list | PASS: last 3 market days (12/28, 1/4, 1/11), +25%. Flags pack > on hand: Sample Tea C (both sizes, 0 on hand), Sample Balm E (0). Items with no active variant: Old Sample Item F (renamed), Old Sample Item G. |
| sold-through | PASS: no packed/left counts given, so it asks; POS-sold side comes from market-day-sales. |
| restock | PASS: math runs; batch sizes and upcoming-event count are asked for, not guessed. |
| event-profit 1/11 | PASS: 480 − 14.00 fees − 40 booth = $426.00. COGS partial $120.00 (3 sold variants have null unitCost, listed and asked). Fees were present, so the ask-for-rate fallback was not needed. Profit/hour needs booth hours. |
| event-log | PASS: sample `test-out/events.csv` written with only the given fields. |

Fixes: join sales to stock on `variant{id}` (renamed products broke title matching); the tool has its own `first`/`after` args (default 10); UTC day window; paginate productVariants; null `unitCost` handling; profit/hour uses booth hours, not sale times.
