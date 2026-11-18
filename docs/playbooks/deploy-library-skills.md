# Playbook: deploy library skills

> Before writing a custom skill, check whether the library already has it.
> Library facts are in
> [`../PRODUCT_FACTS.md`](../PRODUCT_FACTS.md#the-skill-library).

## Step 1: Inventory what exists

- **Workspace library:** `manage_skills → list` with `targetScope:
  "workspace_library"`. These are your Managed and Template skills, with
  their current assignments (`get` → `assignedAssistantIds`).
- **Day AI's templates:** the 18 GTM templates (Meeting Prep, Deal Brief,
  Pipeline Review, Pipeline Dashboard, Follow-Up Sweep, CRM Cleanup, Record
  Check, Customer Health, …), plus the older Actions, Opportunity Automation,
  Call Coaching, and Deal Coaching. They're well-built, maintained by Day AI,
  and already written to handle a half-set-up workspace.
- **Per agent:** `manage_skills → list` with `targetAssistantId`, so you don't
  add a duplicate slash command (Day AI won't add a template whose slash
  command the agent already uses).

## Step 2: Decide per capability

| Situation | Use |
|-----------|-----|
| A Day AI template does the job as written | **The template.** Someone adds it to each agent in Day AI (Skill Library → **Add to agent**). The harness can't push Day AI templates over the MCP. |
| The team needs the template's job, tailored to its process, identically for everyone | A **Managed** workspace-library skill (`deploymentMode: "SHARED"`), written to the `/write-skill` bar, then `manage_skills → deploy` to the listed agents. Edits to the source sync to every copy. |
| Each person should start from a shared skill and make it their own | A **Template** workspace-library skill (`"TEMPLATE"`), then `deploy`. Each agent gets a fork; later source edits affect only new forks. |
| It must be tailored to one person | A per-agent skill (`targetScope: "agent"`). |

Prefer the template when it's honestly good enough. A Day AI–maintained skill
that runs well beats a custom one that drifts. Write custom skills where the
company's own process, pages, or language must be inside the prompt.

## Step 3: Budget the slots

Every **automated** library skill (template or workspace) uses a slot on
each agent it lands on, checked against that agent's tier. `deploy` returns a
per-agent result (`deployed` / `already_deployed` / `failed`). A slot or
tier failure on one agent doesn't stop the others. Read every result and
report the failures.

Template minimum tiers matter here: Buying Group and Enrich a List need
Executive; Pipeline Dashboard, CRM Cleanup, Record Check, and Anonymize need
Professional.

## Step 4: Preview, deploy, verify

- **Preview:** the skill (or template name), the mode (Managed or Template),
  the target agents, the trigger and channel, and the slot impact per agent.
  Deploy is Admin/Owner only and additive. Agents not listed are never
  touched; confirm the list.
- **For Day AI templates,** the preview is a checklist the operator (or each
  agent's owner) follows in Day AI: which template, on which agent, and with
  which schedule and channel.
- **Verify** from the agent's skill list afterwards, and from `get_history`
  once it has had time to fire. "Deployed" isn't "delivering."

## Maintaining it

- Edit Managed skills at the **source** (the library record), never the
  instance. Instances inherit attachments and prompt from the source.
- Use `promptEdits` with `basedOnPromptVersion` for changes, and
  `list_versions` / `get_version` to see or restore history.
- Before undeploying, read `assignedAssistantIds`. Undeploying a Template
  removes only the deployment entry; existing forks stay with their owners.
