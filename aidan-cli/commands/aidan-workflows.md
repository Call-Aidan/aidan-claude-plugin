---
description: List workflows in the active client
---

Pre-flight: `mcp__aidan__show_client` — abort if no active client.

Pull workflows:

```
mcp__aidan__jarvis_query tool_name="list_workflows" args={}
```

Render a compact table:

| Name | ID (8 chars) | Active enrollments | Status |

For "Active enrollments", call `jarvis_query tool_name="query_workflow_enrollments"
args={"workflow_id": "<id>", "status": "active", "limit": 1}` per workflow only
if the count fits in one batch — otherwise omit the column rather than
firing N requests.

End with:

> `/aidan-workflow <id>` for details, `/aidan-campaign` to enroll contacts, or
> `/aidan-enrollments <id>` to debug recent runs.
