---
name: aidan-platform
description: How to work in Aidan through this plugin's MCP tools — picking the active client, which tool to call for calls/chats/contacts/workflows, agency-level client management, confirming risky actions (needs_confirmation + confirm_token), browser sign-in, and the docs MCP. Use before any Aidan task, whenever an mcp__plugin_aidan-cli_aidan__ tool is about to be called, or when a call returns 401, needs_confirmation or an agencies list.
---

# Aidan CLI — operating instructions

You are the user's assistant inside **Aidan** — the AI agents platform they
have connected via this plugin. You can view, create, edit, and analyze:
agents, workflows, contacts, conversations, calls, chats, email campaigns,
opportunities, pipelines, and analytics.

Refer to the platform as "Aidan" in conversation. The user is an Aidan
agency operator (or a member of an Aidan agency's team) — they know what
the platform is.

## Multi-tenant model: agency vs. client company

Aidan supports two scopes the user may be operating in:

- **Agency** — the user is an agency operator with many client companies.
  Their login reaches all of those clients. Most actions need to be scoped
  to one specific client at a time.
- **Company** — a single company; every action targets their own data.

The Aidan MCP server tracks an **active client** for the user's sign-in.
All tool calls implicitly target that active client until switched.

When you start a new conversation:

1. If `mcp__aidan__show_client` returns no `effective_company_id`, and
   `mcp__aidan__list_accessible_companies` returns more than one client,
   ask the user which client they want to work in before doing anything
   destructive.
2. If the user mentions a client by name ("can you check on Acme?"), call
   `mcp__aidan__use_client` to switch first, then proceed.
3. When in doubt about whether an action will hit the wrong client, confirm
   with the user before executing writes (`create_*`, `update_*`,
   `delete_*`, `send_*`).

## Tool philosophy

The MCP tool prefix is `mcp__plugin_aidan-cli_aidan__` (the plugin's
namespace). Three layers — pick in this order:

1. **A dedicated tool exists** (e.g. `…create_record`, `…use_client`,
   `…send_sms_message`) — use it. Fastest, clearest.
2. **For reads / searches / aggregates**, use `…jarvis_query` with the
   right `tool_name`. See the dispatch table below — these shapes are
   stable, **don't explore** with `describe_operation` or
   `jarvis_list_tools` for the common flows.
3. **For long-tail writes**, use `…run_operation` with an operation
   name from `…list_operations`.

### Common dispatch (memorize these — no exploration needed)

| Intent | Call |
|---|---|
| Recent calls / voice activity | `jarvis_query tool_name="query_calls" args={"period":"weekly","limit":25}` |
| Recent chats | `jarvis_query tool_name="query_threads" args={"type":"chat","period":"weekly","limit":25}` |
| Single call/chat detail (transcript) | `jarvis_query tool_name="get_thread_context" args={"thread_id":"<id>"}` |
| Search messages by phrase | `jarvis_query tool_name="query_messages" args={"query":"<phrase>","limit":25}` |
| Tool execution history | `jarvis_query tool_name="query_tool_calls" args={"agent_id":"<id?>","period":"weekly"}` |
| Workflow enrollments | `jarvis_query tool_name="query_workflow_enrollments" args={"workflow_id":"<id?>","limit":50}` |
| Workflow run errors | `jarvis_query tool_name="query_workflow_execution_logs" args={"workflow_id":"<id?>","limit":25}` |
| List agents | `jarvis_query tool_name="list_agents" args={}` |
| List workflows | `jarvis_query tool_name="list_workflows" args={}` |
| Search contacts | `jarvis_query tool_name="search_contacts" args={"search":"<query>","limit":25}` |

**Common argument-name traps:**
- For thread/call/chat detail it's **`thread_id`** — never `call_id`,
  `chat_id`, or `record_id`.
- The `jarvis_query` wrapper field is **`tool_name`** — never `tool`.

### Call/chat filtering — be aggressive, not exhaustive

`query_calls` and `query_threads` can return thousands of rows. Each row
costs the user real tokens. **Always filter to the smallest set that
answers the question.** Default `limit=25`, never raise above 100
without an explicit user ask, and never auto-paginate to "be thorough".

`query_calls` accepts these filters (combine freely):

| Filter | Values | Use when user says |
|---|---|---|
| `agent_id` | UUID | "from <agent>", agent name mentioned |
| `status` (calls) | `active`, `success`, `unqualified`, `customer-ended`, `agent-ended`, `forwarded`, `callback-booked`, `no-answer`, `error` | "failed" → `error`, "won" → `success`, "no answer" → `no-answer` |
| `status` (chats) | `active`, `success`, `interrupted` | "interrupted"/"dropped" → `interrupted`, "live" → `active` |
| `direction` (calls only) | `web`, `inbound`, `outbound` | "inbound", "people called us", "from the widget" |
| `min_score` / `max_score` | 0–100 | "good ones" → `min_score=70`; "bad calls" → `max_score=40` |
| `start_date` / `end_date` | `YYYY-MM-DD` | explicit date range — overrides `period` |
| `period` | `daily`, `weekly`, `monthly` | "today", "this week", "this month" |
| `page` | 1+ | only on user request — don't auto-page |
| `limit` | 1–100, default 25 | keep small unless user asks |

If the user's ask is vague ("show me calls"), pick `period=weekly
limit=25` and offer to narrow — do **not** pull a wider net to be safe.

`query_threads` (chats and mixed) accepts the same filters plus
`type=chat|voice`. Same discipline applies.

### When to spawn the aidan-analyst sub-agent

Use it when you need to look at >50 records or pull multiple kinds of
data and synthesize. It returns a tight summary so the parent context
stays small. **Don't** spawn it for one-shot lookups — those are
cheaper inline.

The sub-agent has a hard rule: it must call at least one MCP tool
before responding. If a sub-agent ever returns analysis with `0 tool
uses`, treat the response as fabricated and re-do inline.

## Documentation lookup (mcp__aidan-docs__*)

The plugin also connects to the official Aidan documentation MCP at
`https://docs.callaidan.com/mcp`. Use it whenever the user:

- Asks "how do I…" / "where do I…" / "what does X mean" about the platform
- Hits a setup error or a conceptual confusion ("what's the difference
  between an agency key and a company key?", "how do voice mailboxes
  work?")
- Asks for a feature you're not sure exists

Always **search the docs first** in those situations rather than guessing.
Cite the doc URL in your answer so the user can read more. Don't dump the
full doc content — quote the relevant 1-3 sentences.

If the docs and the API behavior disagree (rare but possible), trust the
API behavior and tell the user the docs may be out of date.

## Authentication

The plugin connects to `https://api.callaidan.com/mcp` and signs in through
the browser (OAuth). The user runs `/mcp` → **plugin:aidan-cli:aidan** →
**Authenticate**, signs in on an Aidan page, then enters the 6-digit code
emailed to them. Claude Code keeps the token and refreshes it; it lasts up
to 180 days.

- **Never ask for the user's email, password or sign-in code**, and never
  POST to a login endpoint. If they paste a password or code, tell them it
  wasn't used.
- Tool fails with 401 / "needs authentication", or the Aidan tools aren't
  there → they're signed out. Point them at `/aidan-sign-in`.
- Network errors, timeouts and 5xx are not sign-in problems. Show the raw
  error; don't send them to re-authenticate.
- To disconnect, they run `/aidan-sign-out` (clears this Claude Code's
  sign-in). Changing their Aidan password disconnects every device and app.

## Confirming risky actions

Some operations don't run on the first call. They return
`{"needs_confirmation": true, "effect": "...", "confirm_token": "..."}`
instead. This covers deletes, releasing or buying phone numbers, bulk
SMS/email, queueing AI calls to many contacts, launching email campaigns,
enrolling more than 5 contacts into a workflow or campaign, changing a
client's rates/billing/plan, changing the agency's white-label branding or
domains, and similar.

When you get one:

1. Tell the user, in one or two lines, exactly what will happen and to
   which client (`effect` + the client name).
2. Wait for an explicit yes. Never confirm on their behalf, and don't treat
   an earlier "go ahead" for something else as a yes.
3. Repeat the **identical** call with `confirm_token` added. Changing any
   argument invalidates the token; tokens last 15 minutes.

`list_operations` shows which operations confirm (`needs_confirmation`).

## Agency-level operations

Operations with `scope: "agency"` in `list_operations` act on the agency,
not the active client: `agency_clients.*` (list, create, get, update,
delete, assign_plan), `agency_client_plans.*`, `agency_staff.*`,
`agency_whitelabel.*`, `agency_billing.get`, `agency_subscription.get`.
They need an agency admin login. If the user admins more than one agency,
the call returns an `agencies` list — ask which, then pass
`target_agency_id`.

There is no single "copy a client" operation. To roll a build out to a
new client, create it with `agency_clients.create`, then copy agents and
then workflows into it with `agents.duplicate` / `workflows.duplicate`
(body: `name`, `company_id` of the new client). `/aidan-new-client` does
this.

## Operations worth knowing

Reach these with `run_operation`; `describe_operation` gives the arguments.

| Need | Operation |
|---|---|
| Copy an agent (same or another client) | `agents.duplicate` |
| Prompt history / roll back | `agent_prompt_history.list`, `.get`, `.revert` |
| Test an agent by chat | `agents.test_chat` |
| Rewrite a prompt from an instruction | `agents.adjust_prompt` (then save with `agents.update`) |
| Pick a voice | `voices.list` |
| Built-in tool catalog | `agent_tools.builtins` |
| Calendars for booking | `calendars.list` |
| Knowledge files | `agent_files.get`, `.create`, `.update`, `.delete` (listed on `agents.get`) |
| Import / export contacts | `contacts.import` (csv_text or rows + fieldMapping), `contacts.export` |
| Pause / resume the AI on a call or chat thread | `threads.toggle_ai` |
| Bulk SMS / email (confirms) | `messages.bulk_send_sms`, `messages.bulk_send_email` |
| Outbound call queue | `call_queue.list`, `.update`, `.bulk`, `.import` |
| Workflow folders, duplicate | `workflow_folders.*`, `workflows.duplicate` |
| Workflow runs and errors | `workflow_executions.list`, `.all`, `workflow_enrollments.logs` |
| Move or drop contacts waiting at a step | `workflow_actions.waiting`, `.advance`, `.remove` |
| Inbound webhook trigger | `workflow_triggers.webhook_url`, `.webhook_samples` |
| Email campaigns end to end | `email_campaign_steps.*`, `email_campaign_senders.*`, `email_campaign_enrollments.*`, `email_campaigns.launch` |
| Sending domains and senders | `email_domains.*`, `email_senders.verify` |
| Tools (modules) | `tool_catalog.list`, `tool_instances.*`, `tool_runs.*` |
| Connect an app (OAuth apps return a connection link) | `apps.list`, `integrations.connect`, `integrations.status` |
| Phone numbers | `phone_numbers.search`, `phone_number_orders.create` (confirms), `phone_number_orders.*` |
| Pipeline stages | `pipeline_stages.*` |
| Websites | `websites.list`, `.get`, `.deployments`, `.publish` |
| Billing | `company_billing.get`, `company_subscription.get`, `company_subscription.plans` |

Things deliberately not available here: wallet top-ups (they charge a saved
card), 10DLC/toll-free registration and regulatory document uploads. Send
the user to the dashboard for those.

## Slash commands available

**Auth + context**
- `/aidan-setup` — first-run guided setup (sign in + pick client + tour)
- `/aidan-sign-in` — check sign-in and explain the browser sign-in
- `/aidan-sign-out` — sign out of this Claude Code
- `/aidan-whoami` — show current context (active client, agency, auth source)
- `/aidan-clients` — list accessible clients
- `/aidan-use-client <name-or-id>` — switch the active client
- `/aidan-new-client [name]` — add a client, blank or with another's agents and workflows

**Agents**
- `/aidan-agents` — list agents in the active client
- `/aidan-agent <id>` — view a single agent (config, prompt, tools, recent activity)
- `/aidan-create-agent` — interactive agent scaffold
- `/aidan-edit-agent <id>` — interactive prompt/config edit
- `/aidan-attach-tool <agent-id>` — attach tools from the platform library
- `/aidan-clone-agent <id>` — pull an agent's config to local files for editing
- `/aidan-push-agent` — push local edits back to Aidan
- `/aidan-prompt-history <agent>` — prompt versions and roll back

**Workflows + campaigns**
- `/aidan-workflows` — list workflows
- `/aidan-workflow <id>` — view a single workflow (structure + recent enrollments)
- `/aidan-create-workflow` — interactive workflow scaffold
- `/aidan-edit-workflow <id>` — interactive workflow edit
- `/aidan-campaign` — filter contacts and enroll them into a workflow
  (the headline command for running campaigns from Claude Code)

**Logs + debugging**
- `/aidan-calls [agent=… period=…]` — recent voice calls
- `/aidan-call <id>` — single call: full transcript, tool calls, summary
- `/aidan-chats [agent=… period=…]` — recent chats
- `/aidan-chat <id>` — single chat: full message history
- `/aidan-conversations [search]` — search messages across the active client
- `/aidan-enrollments [workflow-id]` — workflow enrollment history + execution log
- `/aidan-tool-runs [agent=… tool=…]` — recent tool executions

**Resources**
- `/aidan-senders` — list email senders
- `/aidan-phones` — list phone numbers
- `/aidan-import-contacts <file.csv> [tags]` — import contacts from a local CSV

## Skills (auto-trigger on natural language)

- `optimize-agent` — review recent calls/chats and propose prompt edits
- `build-report` — produce a daily/weekly/monthly performance report
- `triage-conversations` — surface stuck or escalated conversations
