---
name: implement
description: Turn the plan and audit into real configuration in Day AI — invite the right people at the right roles, tune each teammate's agent identity, and deploy role-specific skills. Always previews before it writes. Requires Owner/Admin. Usage: /implement [scope: a person, team, outcome, or "the high-priority audit items"]
---

# /implement

Make the workspace reflect the plan. This is the only skill that writes to your live Day AI workspace — it invites real people, edits teammates' agents, and deploys skills that will email and Slack them. It **always previews the full change set and waits for your approval** before writing.

$ARGUMENTS

Scope the run to what the argument names (a person, a team, an outcome, or "the high-priority items from the last audit"). If no scope is given, propose one from the latest audit and confirm it before proceeding.

---

## Step 1 — Confirm role and read the inputs

1. Call `manage_workspace_members` → `list_configuration` and check `currentUser.roleName`.
   - **Not Owner/Admin** → stop for anything cross-agent or member-related. Tell the operator they can only configure their *own* agent; everything else needs Admin/Owner. Offer to proceed with just their own agent, or to stop.
2. Read the plan (`planning/*.md`, `workspace/PEOPLE.md`), any **initiative(s)** the scope serves (`initiatives/*.md` — most changes advance the success criteria of one), and the relevant recommendations in `rollouts/`:
   - `<date>-agent-audit/REPORT.md` — recommended invites (and the draft nudge emails), missing agents, and weak skills/identities from `/agent-audit`.
   - `<date>-audit/REPORT.md` — plan-vs-workspace gaps from `/audit`.
   - `<date>-<person>-<archetype>/AGENT.md` — deployment-ready agent specs from `/design-agent`.

   If there's no recent audit or spec for the scope, run `/agent-audit` or `/audit` first (or spawn one inline) — don't implement blind. Note which initiative's criteria this change set moves; you'll report progress against it at the end.

---

## Step 2 — Draft the full change set

Spawn the **`agent-implementor`** subagent in implement-mode-draft (no writes yet):

```
Implement task — DRAFT ONLY, do not deploy. Scope: {scope}.

Here is the plan (planning/*.md, workspace/PEOPLE.md) and the latest audit:

{paste audit REPORT.md or the relevant gaps}

Draft the complete change set for this scope: the invite list (who, role, why), agent identity edits
(from → to), and full skill prompts written to write-skill quality (read .claude/skills/write-skill/SKILL.md
first). Ground every skill in the teammate's real role and the workspace's actual data. Respect each
target agent's tier budget for automated skills. Return the full proposed change set in your report
format. Make NO calls that write — propose only.
```

---

## Step 3 — Preview and approve

Present the full change set to the operator, organized as:

- **Invites** — table of email, role, and the plan reference justifying each.
- **Agent identity edits** — per person, the from → to for each field.
- **Skills** — for each: target agent, scope (agent vs. workspace_library), trigger, channel, the plan outcome it serves, and the **full prompt text**. Show enough that the operator can read what their teammate will actually receive.
- **Cost** — what this change set costs: new agents and the seats they need, any tier bump required to fit the automated skills, and which items are free (re-engages, identity rewrites, skills on existing agents). Surface `navigate_to_billing` if a seat is needed. The operator should see the bill before approving, not discover it at deploy time.

Then stop and ask for approval. Make the stakes explicit: invites send email to real people; skills will deliver to teammates on a schedule; new agents and skills consume seats and tier budget. Let the operator approve all, approve a subset, or send back edits. **Do not write anything until they approve.**

---

## Step 4 — Deploy

On approval, spawn the **`agent-implementor`** to execute the approved set:

```
Deploy the approved change set below. Use invite_member for invites, assistant_settings update for
identity, manage_skills create/update for skills (with targetAssistantId for teammates' agents).
Confirm each call succeeded. After deploying, capture the resulting assistant_settings read and
manage_skills list snapshots to rollouts/{date}-{slug}/ so the change is diffable and restorable.
Return the deployed table with results and snapshot paths.

Approved change set:
{the approved subset}
```

Deploy invites and agent/skill changes. Confirm each result; if any call fails (permission, tier budget, validation), report it and continue with the rest rather than aborting the whole run.

---

## Step 5 — Confirm and record

Save the rollout record to `rollouts/<YYYY-MM-DD>-<slug>/` (proposed set, approved set, deployed results, and the post-deploy snapshots). Then summarize:

```markdown
## Rollout complete — {date}

### Deployed
| Change | Target | Result |
|--------|--------|--------|
| invite | grace@acme.com (Member) | sent |
| skill: daily follow-up | Pici's agent | created — first fires Mon 8am ET |

### When skills first fire
{For each scheduled skill: the next fire time.}

### Not deployed
| Item | Reason |
|------|--------|
| ... | over tier budget / permission / sent back for edits |

### Gaps skills can't solve
- {missing integrations, seats needed (navigate_to_billing), onboarding for new invitees}

### Initiative progress
- {Which initiative(s) this advanced, and how its success criteria stand now — e.g. "bootstrap-day-ai: criterion 4 (agent coverage) now met for 5/6 sellers." Don't mark an initiative SUCCEEDED here — that's verified in `/start`, where criteria are checked against the workspace, not assumed from a deploy.}

Snapshots saved to rollouts/{date}-{slug}/. Re-run `/start` to re-take-stock and update initiative status, or `/audit` to re-check plan ↔ workspace alignment.
```

---

## Notes

- **Preview is mandatory.** Never write to the workspace before the operator approves the change set. The only exception is if the operator explicitly says "deploy without preview."
- **Invites are outward-facing.** They email real people. Treat the invite list with the same care as the skill prompts — confirm role and rationale for each.
- **Respect tier budgets.** Automated skills consume the target agent's slots; over-budget creates are rejected. Prefer one excellent scheduled skill over several thin ones, and prefer a single MANAGED workspace-library skill when a whole team needs the same capability.
- **Don't touch Actions or Opportunity-automation skills** unless the scope explicitly includes them.
- **Always snapshot after deploying.** The `read`/`list` output is the only before/after record an agent has.
- If the operator is a Member, the only writes available are to their own agent. Be clear about that rather than letting calls fail one by one.
