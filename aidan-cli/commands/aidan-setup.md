---
description: First-run guided setup — sign in, pick a client, take a quick tour
---

You're walking the user through their first 60 seconds with the Aidan
plugin. Goal: get them from "just installed" to "did something useful"
with zero friction.

Be warm, not chatty. Each step should fit on one screen. If they already
appear to be set up (e.g. signed in + active client), skip ahead.

### Step 0 — quick welcome

Open with one short message:

> Welcome to Aidan CLI. I'll get you set up in under a minute. Three
> quick steps: sign in, pick a client, and a 30-second tour.

### Step 1 — check sign-in status

Try `mcp__aidan__show_client`. Inspect the response carefully — there are
**four** distinct outcomes and only ONE of them should trigger sign-in.
Misclassifying a transport failure as a 401 is what causes the dreaded
re-auth loop, so be strict:

**A. Explicit auth failure** — the response body contains an error code
of `MISSING_AUTH`, `INVALID_TOKEN`, `EXPIRED_TOKEN`, or an HTTP 401 with
a JSON body that names auth as the cause. Only in this case:
> First, let's sign you in. I'll ask for your email and password — they go
> directly to Aidan to issue you a session token.

Then run the same flow as `/aidan-sign-in` (which is itself idempotent —
it will reuse a valid existing token if one is already on disk).

After sign-in completes, run `/reload-plugins` and **retry
`mcp__aidan__show_client` once**. If it now succeeds, continue to Step 2
in the same conversation — do NOT ask the user to restart. If it still
fails, surface the raw error and stop. Do not re-run sign-in a second
time under any circumstances.

**B. MCP transport failure** — the tool itself errored (server didn't
respond, shim crashed, `mcp-remote` couldn't start, network unreachable,
HTML error page returned, JSON parse failure, etc.). This is **not** an
auth problem. Do **not** trigger sign-in. Instead, surface the raw error
to the user and stop:
> The Aidan MCP bridge isn't responding. This isn't an auth problem —
> signing in again won't help and may make it worse. Here's the raw
> error: `<error>`. Try `/reload-plugins`; if that doesn't help, fully
> restart Claude Code (`/exit` then `claude`).

**C. 200 with no `effective_company_id`** — signed in but no client picked.
> You're signed in. Now let's pick a client to work with.

Skip to Step 2.

**D. 200 with `effective_company_id` set** — fully set up.
> You're already signed in to **<client name>**. Skipping ahead to the tour.

Skip to Step 3.

**Loop guard.** If you have already attempted sign-in once in this
conversation, do NOT attempt it a second time regardless of what
`show_client` returns. Surface the error and ask the user to investigate
manually. Re-running sign-in mints a fresh session token and invalidates
the one the bridge currently holds — that's the loop.

### Step 2 — pick an active client

Call `mcp__aidan__list_accessible_companies`. Three cases:

**A. Single-company key (no agency context).** The list call will fail
or return only one company. Tell them:
> Your account has access to one company: **<name>**. That's already your
> active context — no switching needed.

**B. Multiple clients (agency).** Render a compact list:

```
You have access to <N> clients:

  1. Acme Roofing
  2. Beta Dental
  3. Gamma Plumbing
  4. Delta Realty
  ...

Which one would you like to start with? (Type the name or number — you
can switch any time with `/aidan-use-client <name>`.)
```

Wait for their answer. On reply:
- If they typed a number, map to the indexed name.
- Call `mcp__aidan__use_client identifier="<name>"`.
- Confirm: `Active client → **<Name>**.`

**C. No clients found.** Surface the error and stop — they need to set up
their account first. Don't continue the tour.

### Step 3 — 30-second tour

Once a client is active, give them a single message with the most
valuable next steps:

```
You're set up. Here's what to try first:

**See what's there**
- `/aidan-agents` — list your agents
- `/aidan-workflows` — list your workflows
- `/aidan-calls period=weekly` — recent calls with summaries

**Build something new**
- `/aidan-create-agent` — interactive agent scaffold
- `/aidan-create-workflow` — interactive workflow scaffold

**The headline command**
- `/aidan-campaign` — filter contacts and enroll them into a workflow
  ("add 500 unreached contacts with phone numbers to the AI Reactivation
  Sequence")

**Or just talk to me in plain English** — I'll route to the right tools.
Try: "show me last week's report" or "which agents are failing on
objections?"

If you have multiple clients, switch with `/aidan-use-client <name>`
any time. Run `/aidan-whoami` to see who you're working as right now.
```

End there. Don't ask "what would you like to do next?" — let them drive.

### Guardrails

- **Don't run any of the tour commands automatically.** This is just a
  briefing — the user picks what to do next.
- **Don't loop back to Step 1 if Step 2 fails.** A failure in client
  pickup means something's wrong with their account, not the auth.
- If `mcp__aidan__show_client` itself errors (network, MCP unreachable),
  surface the raw error — they need to know the plugin can't reach the
  Aidan server, and that's a different problem from sign-in.
