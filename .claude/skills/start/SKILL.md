---
name: start
description: The entrypoint for GTM Brain. Verifies the Day AI MCP connection and the operator's role, takes stock of every initiative in initiatives/, reports progress against each one's verifiable success criteria, and kicks off the agents and skills the active initiatives need. On a fresh clone it runs the default bootstrap initiative (connect, identify people, scaffold the plan). Run this whenever you sit down to work. Usage: /start [an initiative slug or focus]
---

# /start

The front door — every time, not just the first time. `/start` reads your initiatives, tells you where each one stands against its own definition of success, and kicks off whatever the active ones need next. On a fresh clone, the only initiative is `bootstrap-day-ai`, so `/start` behaves like first-run setup: connect, learn who's in the workspace, scaffold the plan.

$ARGUMENTS

If arguments name an initiative (a slug) or a focus, scope the run to that. Otherwise take stock of everything.

By the end of a `/start` run the operator should have: a confirmed connection and role, a clear **status board** of every initiative, an honest read of progress against each one's `success_criteria` (verified from the workspace, not assumed), and a kicked-off or clearly-recommended next move for each active initiative.

---

## Step 1 — Verify the connection and the role

Hard gate. Do it first, every run.

Call `manage_workspace_members` with `action: "list_configuration"`.

- **If the call fails** (no MCP, not authorized): stop. Tell the operator the `day-ai` MCP server isn't connected or authorized, and point them at the README's "Connect the Day AI MCP server" section. Don't proceed.
- **If it succeeds**, read `currentUser.roleName` and report it plainly:
  - **Owner or Admin** → "You're an {Owner/Admin}. You can do everything in this harness: read and edit any teammate's agent, create skills for them, and manage members." Proceed.
  - **Member** → "You're a Member. You can build the planning layer, drive initiatives, and configure *your own* agent, but inviting people, editing teammates' agents, and creating skills for others require Admin or Owner — those steps will be blocked." Proceed with expectations set.

Report the basics: workspace name, member count by role, claimed domains.

---

## Step 2 — Take stock of the initiatives

Read every `initiatives/*.md` (skip `README.md` and `TEMPLATE.md`). Parse the frontmatter of each.

**If `initiatives/` has no initiative files** (someone deleted the default), recreate `bootstrap-day-ai.md` from `initiatives/TEMPLATE.md` + the spec in `initiatives/README.md`, then continue. There is always at least one initiative.

For each initiative, **verify progress against its `success_criteria` from the workspace, not from the file's status field** — this is the core principle (confirm states from outcomes, not config). Pull what each criterion needs:

- agent-coverage criteria → `assistant_settings → mode: "list"`, mapped to members
- "skill firing / delivering" criteria → `manage_skills → list` then `manage_skills → get_history` (read the run's `notification.delivered` boolean for delivery; never infer from a schedule existing)
- people/roster criteria → `list_configuration` + `list_suggested_invites`
- plan/Pages criteria → the files in `planning/` and the Pages tools
- pipeline/forecast criteria → `search_objects`
- shared-repo criteria → check `git remote -v` (origin points at the team's own repo, **not** `day-ai/gtm-brain` the template) and that it's a private repo with the operators as collaborators (`gh repo view --json visibility,name` and `gh api repos/{owner}/{repo}/collaborators` if `gh` is available; otherwise ask the operator). A still-on-the-template remote means criterion 1 isn't met — point them at the README's "Your team's GTM Brain repo."

Spawn the **`data-analyst`** as a grounding subagent when a set of criteria needs real evaluation (agent coverage, skill delivery) — ask for the factual snapshot, not the full opinionated report. Keep it to what the criteria require.

Present the **status board**:

```markdown
## Initiative board — {date}

| Initiative | Status | DRI | Target | Progress (criteria met) | Next move |
|------------|--------|-----|--------|-------------------------|-----------|
| Get Day AI set up and delivering value | IN_PROGRESS | — (no DRI) | 2026-07-21 (42d) | 3/6 — coverage + delivery open | /agent-audit, then /design-agent |
| {…} | NEW | jordan@… | 2026-08-01 | 0/4 — not started | kick off |
```

Call out, briefly: any initiative with **no DRI**, any **past its target** still open, anything **PAUSED** with a stale log, and any criterion you **couldn't verify** (permission-gated or needs a tool that doesn't exist — point at `docs/MCP_REQUIREMENTS.md`).

---

## Step 3 — Kick off what the active initiatives need

For each `NEW` or `IN_PROGRESS` initiative in scope, read its **Plan of attack** and drive the next unmet step. Don't redo what's already done — act on the gap between its `success_criteria` and reality.

- **`NEW`** → confirm with the operator, then start it. Propose advancing status to `IN_PROGRESS` (don't change it silently).
- **`IN_PROGRESS`** → run or recommend the next step from its plan. Most steps hand off to another skill — name it and, where it's a read-only/grounding step, run it inline:
  - needs people identified or the plan scaffolded → spawn `gtm-strategist` (as the bootstrap initiative's plan describes)
  - needs the agent-value gap and a costed path → `/agent-audit`
  - needs an agent designed → `/design-agent {person} {archetype}`
  - needs the plan vs. workspace gap → `/audit`
  - needs changes deployed (invites, identities, skills) → `/implement` *(previews first; Owner/Admin)*
  - needs the plan published → `/sync-pages`
- **`PAUSED` / `CANCELLED` / `SUCCEEDED`** → don't act. For `SUCCEEDED`, if your Step-2 verification shows a criterion is no longer true, flag the regression.

**Never write to the workspace from `/start` directly.** `/start` orchestrates and reads; the skills it hands off to (`/implement` especially) own their own preview-then-approve gate. Deploying changes always goes through `/implement`.

**Status changes are proposed, not silent.** When an initiative's criteria all verify true, tell the operator it's ready to mark `SUCCEEDED` and let them confirm. When work begins, propose `NEW → IN_PROGRESS`. Update the frontmatter and append a dated line to the initiative's **Log** only on the operator's go-ahead.

---

## Step 4 — Orient

Close with where things stand and the single best next move:

```markdown
## Where you are

**Workspace:** {name}  ·  **You:** {role} ({what that allows})
**Initiatives:** {N} active · {N} new · {N} paused · {N} done
**Most pressing:** {one line — the initiative nearest its target with the most open criteria, or the highest-leverage next step}

### Do next
- {the single recommended command, e.g. `/agent-audit` to close the coverage + delivery criteria on bootstrap-day-ai}
- Start a new initiative anytime: copy `initiatives/TEMPLATE.md` to `initiatives/<slug>.md`

Your initiatives in `initiatives/` and the plan in `planning/` are yours to edit directly — the agents read them as the source of truth.
```

---

## Notes

- **`/start` reads and orchestrates; it doesn't deploy.** No invites, no agent edits, no skill writes happen here — those go through `/implement`, which previews first. `/start` is safe to run anytime.
- **Verify, don't assume.** An initiative's `status` field is the operator's claim; the `success_criteria` checked against the workspace are the truth. When they disagree, surface it.
- **Respect the cost principle.** When an initiative's plan implies adding agents, carry its value-vs-cost case through — `/agent-audit` and `/implement` total the seat/tier bill before anything deploys.
- **First run = the bootstrap initiative.** On a fresh clone the board has one row (`bootstrap-day-ai`) and `/start` walks its plan: connect → identify people → scaffold the plan, then point at `/agent-audit` and beyond. It should feel like a ~10-minute orientation, not an exhaustive planning session — depth comes later via `/plan`.
- If the operator is a Member, still take stock and report; just be honest that the `/implement` steps in any initiative's plan will be blocked until an Owner promotes them.
