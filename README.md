# Aidan CLI for Claude Code

Run your Aidan agency from the terminal. Build agents, ship workflows,
audit calls, triage conversations, and pull reports — all by just
talking to Claude.

A Claude Code plugin for [Aidan](https://callaidan.com) agency operators
and their teams.

---

## What you can do

Once installed, ask in plain English:

```
> Create a new outbound sales agent for Acme Roofing — friendly,
  qualifies budget and timeline, books a 15-min discovery call.

> Add 500 contacts with a phone number that haven't been called yet in
  the Husband Realty account to the AI Reactivation Sequence workflow.

> Show me last week's report for Beta Dental.

> The booking agent on Gamma Plumbing keeps failing on price objections.
  Pull the last 30 calls and tell me what to fix.

> What conversations need my attention right now?

> Switch me over to Delta Roofing.
```

Or use slash commands directly when you want a specific, repeatable
output format.

### The campaign command

The headline feature: `/aidan-cli:aidan-campaign` is an interactive
campaign builder. It filters contacts in the active client (with full
support for tags, custom fields, last-activity date, has-phone, etc.),
previews the matches, and enrolls them into a workflow in batches. Use
it for reactivation pushes, follow-up sequences, segment-specific
outreach — anything where you'd otherwise be exporting CSVs and
importing into the workflow UI.

---

## Requirements

- [Claude Code](https://docs.claude.com/en/docs/claude-code) v2.x or later
- An [Aidan](https://callaidan.com) account (agency or company), and
  access to its email inbox for the sign-in code.

No Node.js, API key or token file needed.

---

## Install (under 2 minutes)

In Claude Code, run:

```
/plugin marketplace add Call-Aidan/aidan-claude-plugin
/plugin install aidan-cli@aidan
```

Then sign in:

1. Run `/mcp`
2. Choose **plugin:aidan-cli:aidan** and select **Authenticate**
3. Sign in on the Aidan page that opens in your browser, enter the
   6-digit code we email you, then come back

Finish with `/aidan-cli:aidan-setup` to pick a client and take a
30-second tour.

> Your password and code go to Aidan in your browser, never into the
> terminal. Claude Code keeps a token that renews itself and lasts up to
> 180 days. Disconnect any time with `/aidan-cli:aidan-sign-out`, or
> change your Aidan password to disconnect every device.

### Updating from 0.1.x

Version 0.2 replaces the old terminal sign-in. After updating, run `/mcp`
→ **plugin:aidan-cli:aidan** → **Authenticate** once. You can delete
`~/.aidan/credentials.json` and any `AIDAN_TOKEN` line in your shell
profile; nothing reads them any more.

---

## Slash commands

All commands are namespaced under `/aidan-cli:` to avoid collisions
with Claude Code built-ins or other plugins.

**Auth + context**

| Command | What it does |
|---|---|
| `/aidan-cli:aidan-setup` | First-run guided setup — sign in, pick a client, quick tour |
| `/aidan-cli:aidan-sign-in` | Check you're signed in, and how to sign in if not |
| `/aidan-cli:aidan-sign-out` | Sign out of this Claude Code |
| `/aidan-cli:aidan-whoami` | Show current context — active client, agency, auth source |
| `/aidan-cli:aidan-clients` | List every client your account can access |
| `/aidan-cli:aidan-use-client <name>` | Switch the active client |
| `/aidan-cli:aidan-new-client [name]` | Add a client — blank, or with the agents and workflows of one you've built |

**Agents**

| Command | What it does |
|---|---|
| `/aidan-cli:aidan-agents` | List agents in the active client |
| `/aidan-cli:aidan-agent <id>` | View a single agent — config, prompt, tools, recent activity |
| `/aidan-cli:aidan-create-agent` | Interactive new-agent scaffold |
| `/aidan-cli:aidan-edit-agent <id>` | Interactive prompt/config edit |
| `/aidan-cli:aidan-attach-tool <agent-id>` | Attach tools from the platform library |
| `/aidan-cli:aidan-clone-agent <id>` | Pull an agent's config to local files for editing |
| `/aidan-cli:aidan-push-agent` | Push local edits back to Aidan |
| `/aidan-cli:aidan-prompt-history <agent>` | Prompt versions, compare, roll back |

**Workflows + campaigns**

| Command | What it does |
|---|---|
| `/aidan-cli:aidan-workflows` | List workflows |
| `/aidan-cli:aidan-workflow <id>` | View a workflow — structure + recent enrollments |
| `/aidan-cli:aidan-create-workflow` | Interactive new-workflow scaffold |
| `/aidan-cli:aidan-edit-workflow <id>` | Interactive workflow edit |
| `/aidan-cli:aidan-campaign` | Filter contacts and enroll them into a workflow |

**Logs + debugging**

| Command | What it does |
|---|---|
| `/aidan-cli:aidan-calls [agent=… period=…]` | Recent voice calls |
| `/aidan-cli:aidan-call <id>` | Single call: full transcript, tool calls, summary |
| `/aidan-cli:aidan-chats [agent=… period=…]` | Recent chats |
| `/aidan-cli:aidan-chat <id>` | Single chat: full message history |
| `/aidan-cli:aidan-conversations [search]` | Search messages across the active client |
| `/aidan-cli:aidan-enrollments [workflow-id]` | Workflow enrollment history + execution log |
| `/aidan-cli:aidan-tool-runs [agent=… tool=…]` | Recent tool executions |

**Resources**

| Command | What it does |
|---|---|
| `/aidan-cli:aidan-senders` | List email senders |
| `/aidan-cli:aidan-phones` | List phone numbers |
| `/aidan-cli:aidan-import-contacts <file.csv> [tags]` | Import contacts from a local CSV |

---

## Skills (auto-trigger)

Claude triggers these automatically when you describe what you want — no
need to remember a command name.

| Skill | Triggers on |
|---|---|
| `optimize-agent` | "improve this agent", "calls keep failing", "audit performance" |
| `build-report` | "give me a weekly report", "how did we do this month", "summarize last week" |
| `triage-conversations` | "what needs my attention", "any unread", "anything stuck" |

---

## For agency operators

If your account is an **agency account**, signing in gives you access
to every client company under your agency. Switch between them at any
time:

```
/aidan-cli:aidan-use-client acme
```

The selection persists across Claude Code restarts and is kept per
sign-in, so two teammates in the same agency don't step on each other's
active client.

Agency admins can also manage clients themselves from here: add one with
`/aidan-cli:aidan-new-client` (blank, or with the agents and workflows of
a client you've already built), change a client's limits or plan, and manage client plans
and white-label settings. Just ask.

### Using Codex instead?

The same Aidan server works in OpenAI Codex. See
[Connect to Codex](https://docs.callaidan.com/codex-connector).

### Recommended workflow

Keep a folder per client:

```
~/clients/
  acme/
  beta-dental/
  gamma-plumbing/
```

`cd` into a client's folder before launching Claude Code. Use
`/aidan-cli:aidan-use-client` once per folder to set the context;
after that, the client stays active for that session.

---

## Built-in docs lookup

The plugin also wires Claude Code into the official
[Aidan docs](https://docs.callaidan.com) via a separate MCP. Ask any
"how do I…" or "what does X mean" question and Claude searches the docs
automatically.

```
> what's the difference between an agency key and a company key?
> how do I set up a voice mailbox?
> what does "concurrent call limit" mean?
```

---

## Sub-agent

`aidan-analyst` is a read-only sub-agent that Claude can spawn for
deep analysis (hundreds of calls, complex pattern detection, etc.)
without filling up the main conversation context. You don't need to
invoke it directly — Claude calls on it when the analysis is heavy
enough to warrant it.

---

## Troubleshooting

### Aidan tools are missing, or calls say "needs authentication"

You're signed out. Run `/mcp`, choose **plugin:aidan-cli:aidan** and
select **Authenticate**.

If sign-in keeps failing, choose **Clear authentication** in the same
menu, then **Authenticate** again. No code in your inbox? Check spam, or
use **Send a new code** on the sign-in page. Changing your Aidan password
signs you out too.

### "Plugin not found in any marketplace" on install

Make sure you're using the qualified install syntax:

```
/plugin install aidan-cli@aidan
```

Not just `/plugin install aidan-cli`.

### Slash commands aren't appearing

Run `/reload-plugins` once, then type `/aidan-cli:` and you should see
the full list in the autocomplete picker.

---

## Before anything risky

Deleting things, releasing or buying numbers, bulk SMS/email, queueing AI
calls to a list, launching email campaigns, enrolling more than 5 contacts
and changing a client's billing don't run straight away. Aidan sends back
a summary, Claude asks you, and it only runs after you say yes.

---

## Privacy and security

- You sign in on an Aidan page in your browser, with your password and a
  code emailed to you. Neither goes through Claude Code.
- Claude Code holds a token, not your login. It's replaced every hour and
  lasts up to 180 days.
- Claude can do what your Aidan login can do, and no more.
- Disconnect any time with `/aidan-cli:aidan-sign-out`, or by changing
  your password (which disconnects every device).
- Active-client selection is stored server-side per sign-in, so teammates
  sharing an agency don't step on each other's context.
- All traffic goes over HTTPS to the Aidan MCP endpoint. No
  third-party telemetry.

---

## Support

- Docs: [docs.callaidan.com](https://docs.callaidan.com)
- Email: [support@callaidan.com](mailto:support@callaidan.com)
- Issues: open a GitHub issue on this repo

---

## Repo layout

```
.
├── .claude-plugin/marketplace.json    ← Claude Code marketplace manifest
├── aidan-cli/                         ← the plugin
│   ├── .claude-plugin/plugin.json
│   ├── commands/                       ← slash commands
│   ├── skills/                         ← auto-trigger skills (aidan-platform holds the operating instructions)
│   └── agents/                         ← sub-agents
└── README.md
```

---

## License

MIT — see `LICENSE`.
