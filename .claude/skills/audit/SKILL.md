---
name: audit
description: Compare the current Day AI workspace (members, agents, skills) against the planning layer and produce a prioritized gap report. Read-only — makes no changes. Run before /implement. Usage: /audit [a specific person, team, or outcome to focus on]
---

# /audit

Diff the plan against reality. For every outcome the plan calls for, is the workspace actually configured to deliver it? For every teammate, is their agent set up to do their real job, or sitting on defaults? Who's named in the plan but missing from the workspace, or in at the wrong role?

`/audit` makes **no changes**. It produces the gap report that `/implement` then acts on.

$ARGUMENTS

If an argument scopes the run (a person, a team, an outcome), focus there. Otherwise audit the whole workspace against the whole plan.

---

## Step 1 — Read the plan

Read `planning/COMPANY_PLAN.md`, `planning/STRATEGY.md`, `planning/OUTCOMES.md`, and `workspace/PEOPLE.md`. The plan is the spec you're auditing against. If the plan is empty or mostly TODOs, stop and send the operator to `/plan` first — there's nothing to audit against.

---

## Step 2 — Snapshot the workspace

Spawn the **`data-analyst`** subagent:

```
Pull a current workspace snapshot for an audit: people & roles (including pending/declined invites
and suggested invites), full agent/skill configuration coverage (which agents are tuned vs. default,
which have role-specific skills vs. empty/generic), pipeline shape and whether it's maintained, and
activity/coverage. Follow your output format. Flag any permission-gated tools.
```

---

## Step 3 — Diff plan vs. reality

Spawn the **`agent-implementor`** subagent in audit mode:

```
Audit task. Here is the plan (planning/*.md, workspace/PEOPLE.md) and the current workspace snapshot:

{paste data-analyst output}

For each outcome in planning/OUTCOMES.md, determine whether the workspace is configured to deliver it
and what's missing. For each teammate in PEOPLE.md, assess whether their agent identity is tuned and
their skills are role-specific and grounded vs. default/generic. Identify people named in the plan who
aren't in the workspace or are at the wrong role. Produce the gap report with a prioritized list of
proposed changes. Make NO changes.
```

---

## Step 4 — Write the gap report

Save the report to `rollouts/<YYYY-MM-DD>-audit/REPORT.md` and present a summary. Structure:

```markdown
# Workspace Audit — {date}

## Coverage scorecard
| Plan outcome | Owner | Workspace today | Gap | Priority |
|--------------|-------|-----------------|-----|----------|
| AEs never drop a follow-up | Pici | no follow-up skill on any AE agent | deploy daily follow-up skill | High |

## People gaps
| Person (in plan) | In workspace? | Role | Gap |
|------------------|---------------|------|-----|

## Agent & skill gaps
| Person | Agent | Issue | Proposed change |
|--------|-------|-------|-----------------|

## What the data won't support yet
- {outcomes blocked by missing integrations, thin/stale pipeline, seat/tier limits}

## Recommended /implement scope
{The highest-leverage subset to roll out first, in priority order.}
```

---

## Step 5 — Hand off

Close by pointing at implementation, with the recommended first scope:

```markdown
## Audit complete — {N} gaps found ({N} high priority)

Report saved to rollouts/{date}-audit/REPORT.md

### Next
- `/implement {scope}` → roll out the {N} high-priority changes (previews before writing)
- `/plan`              → if the audit revealed the plan itself is wrong or incomplete
```

---

## Notes

- **Read-only. No invites, no agent edits, no skill writes.** Auditing is about seeing clearly before acting.
- Prioritize by leverage: a change that delivers a high-value outcome for a whole team beats a polish pass on one agent.
- If the audit reveals the *plan* is wrong (an outcome nobody can own, a strategy the data contradicts), say so — sometimes the right fix is `/plan`, not `/implement`.
- Distinguish "the workspace isn't configured for this" (an implement gap) from "the data can't support this yet" (an integration/onboarding gap). They have different owners.
