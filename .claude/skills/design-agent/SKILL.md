---
name: design-agent
description: Design one complete, deployment-ready Day AI agent for a person — its identity (name, title, description = its system prompt) and its 1–3 starter skills — built on a proven archetype (CRM Data Nerd, Coach, Relationship Radar, etc.) and grounded in the person's real role and the company's process. The companion to /write-skill (one skill) — this designs a whole human-shaped agent. Usage: /design-agent {person} [archetype or job-slice]
---

# /design-agent

Design a complete agent — a human-shaped job description handed to AI. Where `/write-skill` crafts a single skill, this designs the *whole agent*: who it is, the slice of a real person's week it owns, its identity fields (which *are* its operating definition), and the starter skills that make it real on day one.

Use it after `/agent-audit` recommends an agent, or any time you want to stand up a specific agent well.

$ARGUMENTS

The first argument is the **person** the agent is for (a name/email from `workspace/PEOPLE.md`). The second is the **archetype or job-slice** (e.g. `coach`, `crm-data-nerd`, or a free-form "keeps our partnerships pipeline current"). If either is missing, ask — don't guess who it's for or what it does.

---

## Step 1 — Understand the person and the company's process

Spawn or use the **`data-analyst`** to gather grounding (or read it from a recent `/agent-audit` report):

- The person's real role and daily work, from `workspace/PEOPLE.md` and their activity in the graph (`search_objects`, a recent meeting via `get_meeting_recording_context`).
- The company's **process**, which the agent must be fluent in: the pipeline definition(s) and stage criteria, the sales methodology, any documented process. (Read them via `read_crm_schema` — it covers `native_pipeline` and `native_stage` — and `search_objects` with `includeRelationships` for a pipeline's stages; confirm the interpretation with the operator. Written stage entry/exit criteria may not exist as data anywhere — that part comes from the operator or the playbook docs.)
- What agents the person already has (`assistant_settings` list), so this one covers a *distinct* job slice and doesn't overlap.

A great agent is built on a real slice of this specific person's week — not a generic role template. Anchor on: what does this person do repeatedly, that an agent could own end-to-end?

## Step 2 — Choose and adapt the archetype

Start from the playbook in `.claude/agents/data-analyst.md` (Dimension 2), then tailor hard to the person and company:

- **CRM Data Nerd** — keeps opportunities, notes, and context objects correct and complete from what actually happened in conversations. Works on Day AI or alongside HubSpot/Salesforce.
- **Coach** — deep on every open opportunity; finds patterns, blockers, and what's working on other reps' deals; fluent in *this company's* process; preps before meetings, drafts unblock/follow-up emails.
- **Relationship Radar · Pipeline/Forecast Analyst · Follow-Up Drafter · Market & Account Watch · Chief of Staff · Playbook Editor** — match to the role.

The archetype is the skeleton. The flesh — the company, the person, the pipeline names, the real accounts — is what makes the agent worth having.

**Check the flywheel.** If the agent preps meetings (a Coach especially), look for the living guide pages its prep should read — workspace-shared Pages like a discovery guide (`search_objects`, then `read_page`; the pattern is in `.claude/agents/data-analyst.md`). If a guide exists, the prep skill references it by title and objectId. If none exists, or nobody owns updating it, say so in the spec's **Notes**: the operator likely wants the producer side — an Playbook Editor skill that reviews calls against the guide — designed next, or the flywheel never spins.

## Step 3 — Write the identity (this is the agent's definition)

The identity fields are the agent's operating brief — the description especially is its system prompt, the equivalent of an `.md` agent definition. Write all four to a high bar:

- **First / last name** — a real, human name the person will be glad to work with. Not "Assistant."
- **Title** — the job slice as a real title: "Deal Coach," "Pipeline Data Steward," "Partnerships Watch."
- **Description** — the heart of it. A genuine operating brief: who the agent is, who it works for, what the company does, the slice it owns, how it should behave and communicate, what "great" looks like for it. Specific enough that you could hand it to a new hire. This is where most agents are won or lost.
- **DISC / personality + default language** — matched to the person and the company's culture.

## Step 4 — Design the starter skills

1–3 skills that make the agent real on day one. Read `.claude/skills/write-skill/SKILL.md` and write each prompt to that bar. Favor **push-mode** skills (scheduled or event-triggered, delivered to Slack/email) — an agent whose skills only run when asked isn't automation. For each skill: name, slash command, trigger (with cadence/event + timezone), channel, and the full prompt. Respect the target agent's **tier budget** for automated skills — propose the highest-value one or two, not a pile of thin ones.

## Step 5 — Write the spec

Save a deployment-ready spec to `rollouts/<YYYY-MM-DD>-<person>-<archetype>/AGENT.md`:

```markdown
# Agent Spec — {Agent name} for {Person} ({their role}) at {Company}

**Archetype:** {Coach | CRM Data Nerd | …}   **Owner:** {person email}   **Tier needed:** {…}

## Value vs. cost
- **Value:** {the job slice this agent delegates and the work product it produces proactively — the concrete reason it's worth running}
- **Cost:** {a seat for {Person} (new or existing?); the tier needed to support {N} automated skills; whether that's a tier bump — priced per CLAUDE.md's pricing rules: workspace billing when connected, https://day.ai/pricing otherwise, never from memory}
- **The case:** {one line — why the value clearly clears the cost. If it doesn't, this agent shouldn't be designed; say so and stop.}

## Identity
- **First / last name:** {…}
- **Title:** {…}
- **Description (the agent's operating brief / system prompt):**
  > {the full description, ready to paste into assistant_settings}
- **DISC / personality:** {…}   **Default language:** {…}

## Starter skills
### {Skill name}  ·  /{slash}  ·  {SCHEDULE 0 8 * * 1-5 America/New_York | EVENT … | NEITHER}  ·  {Slack #… | email}
```
{full prompt, ready for manage_skills create}
```
{repeat for each skill}

## Notes
- {what this agent does NOT do — the boundary with the person's other agents}
- {process assumptions to confirm with the operator; data the skills depend on}
```

## Step 6 — Hand off

```markdown
## Agent designed — {Agent name} for {Person}

Spec saved to rollouts/{date}-{person}-{archetype}/AGENT.md

### Next
- Review the description and skill prompts above — this is what {Person} will be working with daily.
- `/implement {person}` → deploy on approval: the operator creates the agent in the Day AI UI from this
  spec's creation card (Settings → Workspace → Agents → New agent; Admin/Owner), then the implementor
  applies identity + skills. Costs a seat and a tier that supports {N} automated skills (see **Value
  vs. cost** above); the implementor surfaces billing (`navigate_to_billing`) before the create so the
  cost is never a surprise.
```

---

## Notes

- **Design only — no writes.** `/implement` deploys on approval — but the agent itself is created manually in the Day AI UI (Settings → Workspace → Agents → New agent; Admin/Owner), following this spec's creation card. The implementor pauses there, verifies the agent exists (`assistant_settings → list`), then applies identity and skills. Agents cannot be created via MCP.
- **One agent = one coherent job slice.** If the job-slice is really two jobs, design two agents — that's the ≥2-agents thesis in action.
- **The description carries the agent.** Spend your effort there. A strong description with two decent skills beats a blank identity with five skills.
- Ground the Coach in the company's actual process. A coach that doesn't know your stages and methodology is a generic motivational poster.
