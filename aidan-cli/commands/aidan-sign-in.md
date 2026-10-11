---
description: Sign in to Aidan in your browser
---

The plugin signs in through the browser. You can't open the sign-in page
yourself, and you must never ask for or handle the user's password or
sign-in code. Your job is to check whether they're signed in and, if not,
tell them the two clicks that do it.

### Step 1 — check

Call `mcp__aidan__show_client`.

- **It returns 200** — they're already signed in. Say so in one line, with
  the active client if there is one, and stop.
- **The tool doesn't exist, or the call fails with 401 / "needs
  authentication"** — go to Step 2.
- **Any other error** (network, timeout, 5xx) — show the raw error and stop.
  It isn't a sign-in problem.

### Step 2 — tell them how

> To sign in:
> 1. Run `/mcp`
> 2. Choose **plugin:aidan-cli:aidan** and select **Authenticate**
>
> Your browser opens an Aidan sign-in page. Sign in with your Aidan email
> and password there, then enter the 6-digit code we email you, and come
> back here. Your password and code go to Aidan, never into this terminal.
>
> No email? Check spam, or use **Send a new code** on that page.

When they say they're done, call `mcp__aidan__show_client` again and
confirm in one line. If there's no active client and they have several,
point them at `/aidan-use-client <name>`.

### Guardrails

- Never ask for, accept, or send an email, password or sign-in code, and
  never POST to any login endpoint. If the user pastes a password or code
  anyway, tell them not to and that it wasn't used.
- If Authenticate keeps failing, suggest: `/mcp` → **aidan** → **Clear
  authentication**, then **Authenticate** again.
