---
name: agent-audit
description: The agent-value review. Evaluates how much value the workspace is actually getting out of its Day AI agents — who's in and who should be (with draft nudge emails), whether everyone has the agents they should (≥2 each; the right archetypes), and whether each agent's skills and identity are strong. Produces a scored report and a prioritized, ready-to-execute recommendation set. Recommend-only. Usage: /agent-audit [a person, team, or dimension to focus on]
---

# /agent-audit

Answer one question with evidence: **how much value is this team actually getting out of its Day AI agents — and what's the fastest path to a lot more?**

Day AI's agents are GTM automation. A team that's delegated real job functions to two-plus agents per person, each running well-written skills on a schedule, is operating at a completely different level than a team where everyone has one generic chat they occasionally talk to. This skill measures where the team sits and hands you a prioritized path to move up.

It is **recommend-only** — it makes no changes. It produces the report and recommendation set that `/implement` then executes.

$ARGUMENTS

Scope to a person, team, or a single dimension (`members`, `coverage`, `quality`) if named. Otherwise review the whole workspace across all three dimensions.

> **`/agent-audit` vs `/audit`:** `/audit` asks "does the workspace deliver *the plan*?" `/agent-audit` asks "is the workspace using Day AI's agents *well*?" — independent of the plan. Run `/agent-audit` to find unrealized value; run `/audit` to check plan alignment. They complement each other.

---

## Step 1 — Read context

Read `workspace/PEOPLE.md` (the cast) and, if they exist, the planning documents (`planning/*.md`) — they tell you what each person's agents *should* be helping with. Not required, but they sharpen the recommendations.

---

## Step 2 — Run the analyst across all three dimensions

Spawn the **`data-analyst`** subagent in full analysis mode:

```
Agent-value analysis. Scope: {scope}. Here is the cast (workspace/PEOPLE.md) and the plan if present:

{paste PEOPLE.md + any planning context}

Evaluate all three dimensions per your instructions:
1. Membership & activation — who's in, who should be, stalled/declined invites, wrong roles.
   Recommend invites/resends AND draft the personal nudge emails the operator can send themselves.
2. Agent coverage — does every active person have ≥2 agents covering distinct job functions?
   Recommend the missing agents by archetype (every seller needs a CRM Data Nerd and a Coach),
   favoring human-shaped agents built on a real slice of each person's week.
3. Agent quality — for each agent: skill craft (vs write-skill bar), automation fitness (scheduled/
   triggered vs manual, right cadence, delivered), and whether the data the skills depend on exists.
   And identity quality: name, title, and especially the description (the agent's system prompt).
   Be explicit about what you can confirm vs. what needs run-history/engagement data we don't have yet
   (cite docs/MCP_REQUIREMENTS.md).
Recommend; do not execute. Return your full analysis in your output format, prioritized for /implement.
```

---

## Step 3 — Write the report

Save to `rollouts/<YYYY-MM-DD>-agent-audit/REPORT.md` and present a tight summary. Keep the analyst's structure: the headline value story first, then the three dimensions, then the prioritized recommendations and the draft nudge emails. Surface a simple **maturity read** up top so the operator feels the gap:

```markdown
# Agent-Value Audit — {date}

## Maturity
| Signal | This workspace |
|--------|----------------|
| Active people with ≥2 agents | {N}/{N} |
| Sellers with both a Coach and a CRM Data Nerd | {N}/{N} |
| Agents with a strong identity (name + title + description) | {N}/{N} |
| Skills that are well-written AND properly automated | {N}/{N} |
| People who should be in the workspace but aren't | {N} |

**Where this team is:** {one line — e.g. "using Day AI as a shared contact list, not as GTM automation."}
**The single highest-leverage move:** {one line}
```

Then the analyst's full findings, recommendations, and the draft nudge emails verbatim.

---

## Step 4 — Hand off

Close by pointing at execution, with a recommended first scope:

```markdown
## Audit complete

Report saved to rollouts/{date}-agent-audit/REPORT.md

### Recommended first moves
1. {highest-leverage recommendation}
2. ...

### Next
- `/design-agent {person} {archetype}` → turn a recommended agent into a deployment-ready spec
- `/implement {scope}`               → deploy the approved invites, identities, and skills (previews first)
- Send the draft nudge emails above from your own inbox to the people who should be here
```

---

## Notes

- **Recommend-only. No invites, no edits, no skill writes.** This skill sees clearly and proposes; `/implement` acts.
- **The draft nudge emails are a first-class output** — they're often the single highest-ROI thing in the report, because re-engaging an already-invited person is nearly free. Draft them in the operator's voice, specific to each person, short and human.
- **Be honest about the effectiveness gap.** You can confirm an agent is well-built and *should* work; confirming it's actually delivering needs run-history/engagement tools the public MCP doesn't expose yet (`docs/MCP_REQUIREMENTS.md`). Don't claim "delivering value" from configuration alone.
- Prioritize by leverage: standing up Coach + CRM Data Nerd agents for a team of active sellers beats polishing one already-good agent.
- Re-run periodically — the maturity table is most useful as a trend.
