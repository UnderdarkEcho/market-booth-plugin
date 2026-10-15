# Market & Booth

Skills for makers who sell at farmers markets, craft fairs and pop-ups. It works on top of the **official Shopify connector** and **Square's official MCP server** (https://mcp.squareup.com/mcp, OAuth, beta). No custom server, no hosting.

Skills: `pack-list`, `market-day-sales`, `sold-through`, `restock`, `event-profit`, `event-log`.

**Safety:** read-only toward Shopify and Square. The only thing it writes is your own event log (CSV, or a Sheet if you ask).

## Connecting Square
Square's hosted MCP uses OAuth. In some apps, including Grok Bot, it currently fails with **"Invalid redirect URI"** until Square allowlists the app. **Fallback:** create your own Sandbox or Production access token in the Square Developer Dashboard, store it as a secret (never paste it into chat), and run Square's official local server read-only. Step-by-step: [`skills/SQUARE-CONNECT.md`](skills/SQUARE-CONNECT.md). The skills detect a failed Square connection and point you there.

## Marketplace description
Turn POS data into a market-day routine: what to pack, what sold, what's left, what to make next, and whether the booth fee paid off. Works with Shopify POS and Square, even when your booth shares one location with online sales.

## Testing
1. Connect Shopify (and/or Square, sandbox OK).
2. "Log an event: Test Market, <a past date with POS sales>, 9-3, booth fee \$40."
3. "Market-day sales for Test Market": check the totals against Shopify/Square reports, and check that unmatched orders are listed.
4. "Sold-through: packed A=10, B=6; left A=4, B=6": check the counts and the mismatch flag.
5. "Pack list for next Saturday", "Restock in batches of 12", "Was Test Market worth it?"
6. Confirm no write tools were called.

Suggested org handle: `midnightsunmercantile`. License: MIT © Midnight Sun Mercantile.
