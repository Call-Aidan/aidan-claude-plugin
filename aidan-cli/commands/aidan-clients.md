---
description: List all accessible client companies
---

Call `mcp__aidan__list_accessible_companies`. Then:

- If it returns a single company, say their login reaches one company only,
  so there's nothing to switch.
- Otherwise, render a compact table of clients with: **name**, **id**
  (truncated to first 8 chars), and any obvious status field on the record.
- After the table, also call `mcp__aidan__show_client` and indicate which
  row is the currently active one with a `→` marker.
- End with a one-line tip: `Use /aidan-use-client <name> to switch, or /aidan-new-client to add one.`

Keep the output tight — no preamble, no recap.
