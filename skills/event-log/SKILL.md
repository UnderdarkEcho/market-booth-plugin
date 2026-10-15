---
name: event-log
description: Use to keep a list of markets and events (date, hours, venue, booth fee, costs, packed/left counts) as a CSV or Google Sheet that the other skills read.
---
# Event log
- Default: `events.csv` in the user's workspace. Columns: id, name, date, start, end, venue, location_id, provider, booth_fee, other_costs, packed_json, left_json, cash_counted, notes.
- Google Sheet: only if the user asks and a Sheets/Drive connector is linked. Writing to the user's own log is the plugin's only write, and it never writes to Shopify or Square.
- Add or edit a row only when the user gives the details. Never fill gaps. Flag double bookings on the same date.
- Keep venue names private unless the user says they may be public.
