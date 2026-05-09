---
description: List agents in the active client
---

Pre-flight: call `mcp__aidan__show_client`. If no active client, abort and
ask the user to run `/aidan-use-client <name>`.

Pull agents:

```
mcp__aidan__jarvis_query tool_name="list_agents" args={}
```

Render a compact table:

| Name | ID (8 chars) | Type | Status |

Top 25 by recency. After the table, end with one short tip:

> `/aidan-agent <id>` for details, or `/aidan-create-agent` for a new one.

No preamble.
