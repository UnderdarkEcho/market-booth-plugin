# Connecting Square (OAuth, with a personal-token fallback)

## Default: hosted Square MCP (OAuth)
The plugin points at Square's official hosted MCP server, `https://mcp.squareup.com/mcp`, which signs you in with OAuth.

This is the **primary, recommended** way to connect:
1. Connect Square from the plugin and click **Authenticate**.
2. Sign in to Square and approve **read** access.
3. Check it by asking "List my Square locations."

If the Square page says **"Invalid redirect URI"**, the app's OAuth callback isn't allowlisted yet. That's a setup issue on our side (maintainers: see `notes/square-redirect-request.md`), not something you did. Use the secondary fallback below only if OAuth isn't working for you.

## Secondary fallback: your own Square access token (local stdio MCP)
Use this only if OAuth won't connect.
This runs Square's official open-source server (`square-mcp-server`) on your own machine with your own token. It works for Sandbox and Production.

### 1. Create the token in the Square Developer Dashboard
1. Sign in at https://developer.squareup.com/apps with your Square account.
2. Click **+ Create an application** (any name, e.g. "Example Booth App"), or open an existing one.
3. Pick the environment at the top: **Sandbox** to test with fake data, **Production** for your real shop.
4. Open **Credentials**.
   - Sandbox: copy the **Sandbox Access Token**.
   - Production: click **Show** next to **Production Access Token** and copy it.
5. Treat it like a password. A production personal access token has full access to your Square account, so keep it private and use the read-only setting below.

### 2. Store it as a secret (never paste it into chat)
- **Never paste the token into a chat with an assistant**, into a skill, or into any file in this repo. If you've pasted it somewhere, revoke it in the Dashboard (Credentials → *Replace*) and make a new one.
- In Grok Bot or another app with a secrets or environment-variable store, save it there as `SQUARE_ACCESS_TOKEN`.
- Locally, keep it in a file that only you can read, outside any git repo:
  ```bash
  mkdir -p ~/.secrets && chmod 700 ~/.secrets
  # put this one line in ~/.secrets/square.env using an editor, not shell history:
  #   ACCESS_TOKEN=EAAA...
  chmod 600 ~/.secrets/square.env
  ```

### 3. Point the plugin at the local server
Replace the `square` entry in `.mcp.json` (or add a second server) with:
```json
{
  "mcpServers": {
    "square": {
      "command": "bash",
      "args": ["-c", "set -a; . ~/.secrets/square.env; set +a; exec npx -y square-mcp-server start"],
      "env": { "SANDBOX": "true", "DISALLOW_WRITES": "true" }
    }
  }
}
```
- For **Production**, remove `"SANDBOX": "true"`.
- Always keep `"DISALLOW_WRITES": "true"`. Market & Booth only reads from Square.
- The token is loaded from your secret file when the server starts, so it never appears in `.mcp.json`.

### 4. Check it
Ask: "List my Square locations." You should see your location(s). If you get `UNAUTHORIZED`, the token is wrong, revoked, or from the other environment. Check the `SANDBOX` setting.
