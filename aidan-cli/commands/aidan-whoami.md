---
description: Show current Aidan context (active client, agency, auth source)
---

Call `mcp__aidan__show_client` and `mcp__aidan__get_execution_context` in
parallel. Then render a compact summary:

- **Active client**: name + truncated id (or "none — no client selected")
- **Agency**: id (truncated) or "single-company key"
- **Auth source**: from `authorization_source` (request / environment / missing)
- **Backend**: base URL

If `effective_company_id` is empty and the agency has multiple accessible
clients, end with: `→ Run /aidan-use-client <name> to pick one.`

No preamble. Five lines max.
