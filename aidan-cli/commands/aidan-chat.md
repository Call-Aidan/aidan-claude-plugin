---
description: View a single chat — full message history + summary
argument-hint: <thread-id>
---

Same shape as `/aidan-cli:aidan-call` but for chat threads.

### Pre-flight

1. `mcp__plugin_aidan-cli_aidan__show_client`.
2. If `$ARGUMENTS` is empty, list recent chats first via the
   `/aidan-cli:aidan-chats` logic.

### Pull (one tool call — `get_thread_context` already returns messages
inline)

```
mcp__plugin_aidan-cli_aidan__jarvis_query
  tool_name="get_thread_context"
  args={"thread_id": "$ARGUMENTS"}
```

**Important:** the arg name is **`thread_id`** — never `chat_id`,
`call_id`, or `record_id`. The wrapper field is **`tool_name`** —
never `tool`.

### Render

```
# Chat <id-prefix>  ·  <agent>  ·  <date>

**Contact:** <name> (<phone or email>)
**Channel:** <platform if available>
**Status:** <status>

## Summary
<thread.summary>

## Messages
<role>: <message>
...
```
