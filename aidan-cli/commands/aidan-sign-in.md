---
description: Sign in to Aidan with your email and password
---

You're guiding the user through a one-time login so the Aidan MCP
connection works.

### Step 0 — reuse existing session if possible (idempotency check)

**Before asking for any credentials**, check whether a usable session
already exists on disk. Minting a new token unnecessarily will invalidate
the one the running MCP bridge currently holds and put the user into a
re-auth loop.

1. Read `~/.aidan/credentials.json`. If it doesn't exist, skip to Step 1.
2. Parse the JSON. Pull `token` and `expires_at`.
3. If `expires_at` is in the future, validate the token by calling:
   ```
   POST https://api.callaidan.com/mcp
   Authorization: Bearer <token>
   Content-Type: application/json
   Accept: application/json, text/event-stream

   {"jsonrpc":"2.0","id":"check","method":"initialize","params":{
     "protocolVersion":"2024-11-05",
     "capabilities":{},
     "clientInfo":{"name":"aidan-cli-signin-check","version":"1"}
   }}
   ```
   A `200` response with a JSON-RPC `result` (no `error` block) means the
   token is valid.
4. If valid, **stop and report success without minting anything**:
   > You're already signed in as `<email>` (token in
   > `~/.aidan/credentials.json` is valid). No new session needed.
   >
   > If you wanted to force a fresh sign-in (e.g. to switch accounts),
   > run `/aidan-sign-out` first, then `/aidan-sign-in` again.
   Then exit. Do NOT proceed to Step 1.
5. Only if the file is missing, the token is expired, or the validation
   call returns an explicit auth error, continue to Step 1.

If the validation call fails for a non-auth reason (network, 5xx, HTML
error page), do NOT proceed to mint a new token — the new one won't fix
a transport problem. Surface the raw error and stop.

### Step 1 — collect credentials

Ask the user (in two separate messages):

1. First message:
   > What's the email you use to sign in to Aidan?

2. After they answer, ask:
   > And your password? (Just paste it — I'll send it directly to Aidan
   > to swap for a session token, then forget it.)

Important: do NOT echo the password back. Do NOT include it in any
follow-up message. Treat it as write-only input.

### Step 2 — exchange for a session token

Call the login endpoint:

```
POST https://api.callaidan.com/auth/plugin/login/
Content-Type: application/json

{
  "email": "<email>",
  "password": "<password>",
  "label": "Aidan CLI on <hostname>"
}
```

For `<hostname>`, run `hostname` via Bash to get the user's machine name —
this becomes the session label so they can identify it later when revoking.

The response will be:

```json
{
  "session_token": "adn_pls_…",
  "session_id": "<uuid>",
  "label": "Aidan CLI on macbook-pro",
  "expires_at": "2026-10-24T…"
}
```

If the response is 401 / "Invalid credentials", apologize and offer to retry.
Don't loop more than twice — point them at password reset:
`https://app.callaidan.com/forgot_password` (or whichever path applies).

### Step 3 — persist the session token

Save the raw `session_token` (no `Bearer ` prefix — the plugin's MCP
helper adds that itself) to **two** places. The credentials file is the
primary path the plugin reads on every MCP connection; the shell rc is a
backup so command-line tools can also use the same env var.

#### 3a — Credentials file (primary, no restart needed)

Use Bash:

```bash
mkdir -p ~/.aidan && \
  printf '{\n  "token": "<SESSION_TOKEN>",\n  "email": "<EMAIL>",\n  "expires_at": "<EXPIRES_AT>"\n}\n' \
  > ~/.aidan/credentials.json && \
  chmod 600 ~/.aidan/credentials.json
```

Substitute `<SESSION_TOKEN>`, `<EMAIL>`, and `<EXPIRES_AT>` from the
login response. The `chmod 600` keeps the token readable only by the
user.

#### 3b — Shell rc (secondary, picked up on next Claude Code launch)

1. Detect the user's shell from `$SHELL`. Pick the matching rc file:
   - zsh → `~/.zshrc`
   - bash → `~/.bashrc` (macOS: `~/.bash_profile` if it exists)
   - fish → `~/.config/fish/config.fish`
2. Check whether `AIDAN_TOKEN` is already exported in that file.
   - If yes: replace the existing line via `Edit`.
   - If no: append `export AIDAN_TOKEN="<session_token>"` via Bash with
     `>>`. The value should look like `adn_pls_…` with no other prefix.
3. Show the user the line you wrote, **but redact all but the last 4
   characters of the token** (e.g.
   `export AIDAN_TOKEN="adn_pls_…AbCd"`).

#### 3c — Confirm

> Signed in as `<email>`. Token saved to `~/.aidan/credentials.json` and
> `<rc-file>`. Run `/reload-plugins` to pick up the new session — no full
> restart needed.
>
> You can revoke this session anytime from Aidan → Settings → Active
> Sessions, or by running `/aidan-sign-out` in Claude Code.

### Step 4 — offer to verify

> Want me to test the connection?

If they say yes, run `/reload-plugins`, then `mcp__aidan__show_client`.
If `show_client` returns 401 even after `/reload-plugins`, the helper
isn't being invoked — tell the user to fully restart Claude Code as a
last resort.

### Guardrails

- **Never echo or re-display the password.** Once you've sent it to the
  login endpoint, it's gone. Don't write it to any file.
- **Never echo the full session token.** Always redact to last 4 chars.
- **Never send credentials anywhere except** `https://api.callaidan.com/auth/plugin/login/`.
- **Never commit anything.** If the user's shell rc is in a tracked
  dotfiles repo, point that out and let them commit manually.
- If the login keeps failing, suggest password reset rather than retrying
  endlessly.
