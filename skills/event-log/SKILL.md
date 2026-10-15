---
name: event-log
description: Use to keep a list of markets and events (date, hours, venue, booth fee, costs, packed/left counts) as a CSV or Google Sheet that the other skills read.
---
# Event log
- Default: `events.csv` in the user's workspace. Columns: id, name, date, start, end, venue, location_id, provider, booth_fee, other_costs, packed_json, left_json, cash_counted, notes.
- Google Sheet: only if the user asks and a Sheets/Drive connector is linked. Writing to the user's own log is the plugin's only write, and it never writes to Shopify or Square.
- Add or edit a row only when the user gives the details. Never fill gaps. Flag double bookings on the same date.
- Keep venue names private unless the user says they may be public.

## If Square won't connect
If a Square tool call fails with an auth error (`401`, `UNAUTHORIZED`, "not connected", "authentication required"), or the user says Square sign-in showed **"Invalid redirect URI"**, the hosted OAuth connection isn't working. Don't retry over and over. Instead:
1. Tell the user that Square's hosted sign-in doesn't work in this app yet. It's a known Square allowlist issue, not something they did.
2. Point them to `skills/SQUARE-CONNECT.md` → "Fallback: your own Square access token" (Dashboard → app → Credentials → Sandbox or Production token, stored as a secret, local server with `DISALLOW_WRITES=true`).
3. **Never ask the user to paste a token into chat.** If they paste one, don't repeat it or store it. Tell them to revoke it in the Dashboard and save a new one as a secret.
4. Meanwhile, carry on with Shopify data if it's connected, or with numbers the user gives you, and label the Square part "not connected".
