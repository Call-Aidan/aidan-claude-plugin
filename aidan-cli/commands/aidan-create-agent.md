---
description: Interactive scaffold for a new agent
argument-hint: [agent-name]
---

You are creating a new agent for the active client. If `$ARGUMENTS` is empty,
ask the user what to name the agent. Otherwise treat `$ARGUMENTS` as the
proposed name.

### Pre-flight

1. Call `mcp__aidan__show_client`. If `effective_company_id` is empty,
   abort and tell the user to run `/aidan-use-client <name>` first.
2. Call `mcp__aidan__describe_operation` with `operation="agents.create"`
   to see the current required body fields. Use that contract to drive the
   questions below — don't hardcode field names from memory.

### Gather inputs

Ask the user (in one message, as a numbered list):

1. **Purpose** — what does this agent do? (1-2 sentences)
2. **Channel** — call, chat, or both?
3. **Voice / persona** — friendly, professional, blunt, etc.
4. **Initial system prompt** — offer to draft one based on (1)-(3) and have
   them edit, OR let them paste their own.
5. **Tools** — any specific integrations to enable? (skip if unsure)

### Model

Set it yourself rather than asking: `gpt-4.1` for a call agent, `gpt-5.4` for
a chat one.

Offer **GPT Live-1** if they ask for the agent to sound human, or say the one
they have sounds robotic. A pipeline voicebot transcribes, thinks, then
speaks, and the joins are audible; Live-1 hears and speaks directly, so it
interrupts, hesitates and changes tone the way a person does.

| Model | Character |
|---|---|
| `gpt-live-1-terra` | Balanced. The default, and the one to pick unless asked |
| `gpt-live-1-sol` | Deepest reasoning, slower |
| `gpt-live-1-luna` | Fastest and cheapest |

Three things to say before they agree to it:

- It costs **2c/min more** than a pipeline minute, on top of their call rate.
- It takes **two prompts**, sent as `prompt_parts`: `voice` holds everything
  the agent needs to run the call (role, script, FAQs, objections, tone) and
  `backend` holds the tools only — which to run, when, and what each needs.
  Put the personality in `voice`; a Live-1 agent with its script in `backend`
  sounds like it is reading from another room.
- Voices are its own set of 22, `provider: openai-live`, default `ripple`
  (Australian male). Transcriber settings and a separate TTS voice do not
  apply; background sound plays only while the agent is speaking. Only
  `function`, `apiRequest`, `endCall`, `dtmf` and `transferCall` tools are
  accepted.

### Create

Once all inputs are confirmed, call `mcp__aidan__create_record` with
`resource="agents"` and the gathered data. On success:

- Show the new agent's id.
- Offer next steps: `Want me to add a tool, attach a knowledge file, or test it with a call?`

If validation fails, surface the field-level errors and offer to retry.
