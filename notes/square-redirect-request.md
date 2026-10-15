# Internal: add Grok Bot's OAuth redirect URI to our Square app

Square app: **Market & Booth Test**, Application ID `sq0idp-frF18R28Wyj6I9QMosOWoA`

## Redirect URI to add
```
https://www.cursor.com/agents/mcp/oauth/callback
```
Since 2026-09-07, Grok Bot's remote-MCP OAuth sends this HTTPS callback for app and cloud sign-ins (the old `grokbot://mcp/oauth/callback` scheme isn't used anymore). The desktop app can use a loopback callback instead: `http://localhost:8787/callback`.
Source: Cursor forum staff answers, topics 171877 and 171111. We haven't seen this value in a Square error yet, so confirm it in step 6.

## Steps
1. Sign in at https://developer.squareup.com/apps with the account that owns the app.
2. Open **Market & Booth Test** and check that the Application ID is `sq0idp-frF18R28Wyj6I9QMosOWoA`.
3. In the left menu, open **OAuth**.
4. Use the toggle at the top to pick the environment. Do **Sandbox** first, then repeat for **Production** when we're ready.
5. Paste `https://www.cursor.com/agents/mcp/oauth/callback` into **Redirect URL** exactly as written: HTTPS, no trailing slash, no query string. Click **Save**.
   - Square allows only one redirect URL per environment. If the field already holds a URL that something else depends on, stop and decide which one to keep before you overwrite it.
6. Test it: in Grok Bot, connect Square through the plugin. On the Square authorize page, check that the `redirect_uri=` in the address bar decodes to the same URL. If it doesn't, paste that decoded value into step 5 instead.
7. Approve, then ask "List my Square locations." Once Sandbox works, repeat steps 4–7 for Production.

## Caveat
This allowlist only applies to OAuth that uses **our** app's client ID. The plugin's `.mcp.json` currently points at Square's hosted server, `https://mcp.squareup.com/mcp`, which runs OAuth through Square's own client. If the authorize URL in step 6 shows a `client_id` other than `sq0idp-frF18R28Wyj6I9QMosOWoA`, our setting won't affect that flow. In that case, only Square can fix the hosted server's allowlist, and users fall back to the personal-token route in `skills/SQUARE-CONNECT.md`.
