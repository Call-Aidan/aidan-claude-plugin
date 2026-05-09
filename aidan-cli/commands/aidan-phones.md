---
description: List phone numbers available to the active client
---

`mcp__aidan__show_client` — abort if no active client.

Pull phone numbers:

```
mcp__aidan__jarvis_query tool_name="list_phone_numbers" args={}
```

Render:

| Number | Type | Owner (company/agency) | Default for | Capabilities |

`Default for` shows whether this number is the default for SMS, calls,
or both, if that's exposed in the data.

If the user is on an agency key, also include the agency-level numbers
visible to this client.

End with:

> Use one of these IDs in `mcp__aidan__send_sms_message
> phone_number_id="..."` or when configuring a workflow's outbound action.
