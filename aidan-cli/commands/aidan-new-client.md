---
description: Add a new client to your agency — blank, or with the agents and workflows of a client you've already built
argument-hint: [client name]
---

Create a new client company under the user's agency. Client operations act
on the **agency**, not the active client.

### Step 1 — blank or copy?

Ask one question (skip it if `$ARGUMENTS` or the user already said):

> Start the new client blank, or copy the agents and workflows from one
> you've already set up? Copying is the fastest way to roll out the same
> build to another business.

### Step 2 — create the client

Collect, in one message: **business name** (use `$ARGUMENTS` if given),
**contact email**, and **street address, city, state, postcode and
country** (all required). Optionally: website (used for business context)
and a client plan.

If they want a plan, list them first:
`mcp__aidan__run_operation operation="agency_client_plans.list"`.

Then:

```
mcp__aidan__run_operation operation="agency_clients.create" body={
  "name": "<name>", "email": "<email>",
  "address": "<street>", "city": "<city>", "state": "<state>",
  "zip_code": "<postcode>", "country": "<country>",
  "website": "<optional>", "agency_client_plan_id": "<optional>"
}
```

It returns the new client's id. If the result asks which agency (they
admin more than one), show the `agencies` list it returns, ask, and repeat
with `target_agency_id`.

For a blank client, skip to **Finish**.

### Step 3 — copy agents and workflows (copy only)

1. `mcp__aidan__list_accessible_companies` — ask which client to copy from,
   then `mcp__aidan__use_client identifier="<source client>"`.
2. List its agents (`mcp__aidan__jarvis_query tool_name="list_agents"
   args={}`) and workflows (`tool_name="list_workflows"`). Show both and
   ask what to copy; default to everything.
3. Copy the **agents first**, one call each:
   ```
   mcp__aidan__run_operation operation="agents.duplicate"
     path_params={"agent_id": "<id>"}
     body={"name": "<same name>", "company_id": "<new client id>"}
   ```
4. Then the workflows:
   ```
   mcp__aidan__run_operation operation="workflows.duplicate"
     path_params={"workflow_id": "<id>"}
     body={"name": "<same name>", "company_id": "<new client id>"}
   ```
   Workflows find the copied agents by name, so keep the agent names
   unchanged and copy agents before workflows.

Always pass both `name` and `company_id`. Phone numbers, contacts,
connected apps, tools and settings aren't copied; tell the user to set
those up in the new client.

### Finish

Confirm in two lines: the new client's name and id (plus what was copied),
and what's next:

> Created **<name>**. Switch to it with `/aidan-use-client <name>`, then
> `/aidan-create-agent` to build its first agent.

Don't switch the active client automatically — ask first.

### Guardrails

- Creating clients needs an agency admin. A 401/403 means their login isn't
  an agency admin; say so plainly.
- Agencies have a client limit set by their plan. If the create fails with
  "Company limit reached", say so and that upgrading the plan raises it.
