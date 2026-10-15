# DRAFT (not sent): Square developer support, redirect URI allowlist

**To:** Square Developer Support (https://developer.squareup.com/forums or the support form at https://squareup.com/help/contact, developer category)
**Subject:** Allowlist request: Grok Bot OAuth redirect URI for the Square MCP server (mcp.squareup.com)

Hi Square Developer team,

We publish **Market & Booth**, an open-source, read-only plugin for market and pop-up sellers (https://github.com/UnderdarkEcho/market-booth-plugin). It connects to your hosted MCP server at `https://mcp.squareup.com/mcp`.

When users of the Grok Bot app connect Square, the OAuth step fails with **"Invalid redirect URI"**. The same tools work fine through the local `square-mcp-server` with a personal access token, so it seems Grok Bot's OAuth callback just isn't on the allowlist for the hosted MCP's OAuth client.

Could you add this redirect URI to the allowlist?

`<<REDIRECT_URI: copy the redirect_uri= value from the Square sign-in URL that shows the error>>`

If there's a self-serve registration path for MCP clients, such as dynamic client registration or a partner form, we'd be glad to use it. Our plugin only calls read methods (locations, orders.search, inventory, catalog, payments.list).

Thanks,
Richard
Midnight Sun Mercantile (@midnightsunmerc)

---
_Note: the exact redirect URI wasn't in any saved error, log, or config. Before sending, start the Square connect in Grok Bot, copy the `redirect_uri` parameter from the authorize URL in the address bar (URL-decoded), and replace the placeholder._
