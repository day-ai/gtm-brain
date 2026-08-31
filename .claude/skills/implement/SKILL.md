---
name: implement
description: >-
  Turn the plan and audit into real configuration in Day AI — invite the right
  people at the right roles, tune each teammate's agent identity, and deploy
  role-specific skills. Always previews before it writes. Requires Owner/Admin.
  In preflight mode it applies the rollouts/preflight/ payload built during
  map-your-gtm, in dependency order, first-win slice first. Usage: /implement
  [scope: a person, team, outcome, "preflight", or "the high-priority audit
  items"]
metadata:
  internal: true
---

# /implement

Make the workspace reflect the plan. This is the only skill that writes to your live Day AI workspace — it invites real people, edits teammates' agents, and deploys skills that will email and Slack them. It **always previews the full change set and waits for your approval** before writing.

$ARGUMENTS

Scope the run to what the argument names (a person, a team, an outcome, or "the high-priority items from the last audit"). If no scope is given, propose one from the latest audit and confirm it before proceeding.

**Preflight routing.** If the argument is `preflight`, run Preflight mode (below). Otherwise, auto-enter Preflight mode only when all three are true: `rollouts/preflight/` contains payload files beyond its own README.md, no applied record exists (no `rollouts/<date>-preflight/` folder), and the operator didn't name a scope of their own. An explicit non-preflight scope always gets the standard flow; mention the pending preflight payload in the preview instead of overriding what was asked. On a fresh clone, `rollouts/preflight/` holds only its README.md and there is nothing to apply: never enter Preflight mode from that state. If there's also no plan or audit to implement from, don't push ahead blind; stop and send the operator to `/start`, which detects their state and kicks off what actually comes next.

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
- **Pages & folders** — any guide pages or shared folders the set creates or edits: title, folder + sharing, and for updates the targeted before → after. Flywheel deploys create the folder and guide page before the skills that reference them.
- **Workspace instructions** — the full **current text → proposed text**. It's one shared ≤3000-char record and `update` replaces the whole thing, so the operator must see exactly what survives.
- **Cost** — what this change set costs: new agents and the seats they need, any tier bump required to fit the automated skills, and which items are free (re-engages, identity rewrites, skills on existing agents). Surface `navigate_to_billing` if a seat is needed. Price per CLAUDE.md's pricing rules: the workspace's own billing is the source when connected, https://day.ai/pricing otherwise, and the full seats + tiers + slots total is shown, never a unit price. The operator should see the bill before approving, not discover it at deploy time.

Then stop and ask for approval. Make the stakes explicit: invites send email to real people; skills will deliver to teammates on a schedule; new agents and skills consume seats and tier budget. Let the operator approve all, approve a subset, or send back edits. **Do not write anything until they approve.**

---

## Step 4 — Deploy

On approval, spawn the **`agent-implementor`** to execute the approved set:

```
Deploy the approved change set below. Use invite_member for invites, assistant_settings update for
identity, manage_skills create/update for skills (with targetAssistantId for teammates' agents),
create_or_update_folder / create_page / update_page for shared guides (folders and pages before the
skills that reference them), and manage_workspace_instructions update for workspace-wide rules
(list_configuration first, merge into the existing text, write back the complete record).
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

## Preflight mode — applying the map-your-gtm payload

The connect-day flow: everything discovery authored in `rollouts/preflight/` gets applied to the newly connected workspace, in dependency order, with the same preview-then-approve gate at every step. This is the change-management moment; walk it deliberately. The payload formats and full apply order live in `rollouts/preflight/README.md`.

1. **Verify role.** `list_configuration` → `currentUser.roleName` must be Owner or Admin; the payload touches everything. Stop otherwise.
2. **Review the privacy guidance.** Open `workspace/PRIVACY.md` with the operator: confirm the recommended per-persona setup still matches the roster about to be invited, resolve what's resolvable (point them at https://day.ai/trust for Day AI-posture questions), and check each invitee's `ENABLEMENT/privacy-setup-*.md` instructions match the guidance. There is no signature to collect — every user applies their own settings at activation. This step exists so invitees arrive to clear guidance, not a shrug.
3. **Reconcile the map against reality.** The workspace may not be empty (trial data, auto-joined teammates). Diff `INVITES.md` against the live roster, `CUSTOM_PROPERTIES.json` against `read_crm_schema`, and report divergences before proposing anything.
4. **Apply in dependency order, one preview per step:** workspace instruction (merge into the live record, then write back the whole text) → custom properties → pages and folders (the `PAGES/` corpus applies per the canonical algorithm in `rollouts/preflight/README.md`: folders → create pages → per-page image attach and cross-link rewrite, every pageId recorded in `PAGES/.pages-sync.json` as it's created) → invites → **pause for the manual step**: the operator creates each agent in the Day AI UI from its `CREATION_CARD.md`, and you verify each exists via `assistant_settings → list` before continuing → agent identities → skills → imports.
5. **First-win first.** Within every step, items marked `priority: first-win` deploy before the rest. The pilot users' experience on day one is the whole point of having mapped early.
6. **Resolve every `GROUND-AFTER-CONNECT` marker before enabling its skill.** Look up the live value (real stage names, page ids, actual meeting cadence) and write it into the prompt — page ids resolve from `PAGES/.pages-sync.json` first (the pages step just wrote them; a skill and the guide it reads deploy from the same payload, atomically), the graph otherwise. A skill with an unresolved marker does not go live; generic first output is the failure mode we mapped early to avoid.
7. **Hand out the human steps.** Per-user privacy setup instructions and rollout guides from `ENABLEMENT/` go to each person (their activation owner in `PEOPLE.md` verifies completion). These cannot be applied via MCP by design.
8. **Snapshot and record** to `rollouts/<date>-preflight/` exactly like a standard run, and report initiative progress against `map-your-gtm` and `first-win`. Then point at `/start` to verify criteria the normal way.

---

## Notes

- **Preview is mandatory.** Never write to the workspace before the operator approves the change set. The only exception is if the operator explicitly says "deploy without preview."
- **Invites are outward-facing.** They email real people. Treat the invite list with the same care as the skill prompts — confirm role and rationale for each.
- **Respect tier budgets.** Automated skills consume the target agent's slots; over-budget creates are rejected. Prefer one excellent scheduled skill over several thin ones, and prefer a single MANAGED workspace-library skill when a whole team needs the same capability.
- **Don't touch Actions or Opportunity-automation skills** unless the scope explicitly includes them.
- **Always snapshot after deploying.** The `read`/`list` output is the only before/after record an agent has.
- If the operator is a Member, the only writes available are to their own agent. Be clear about that rather than letting calls fail one by one.
