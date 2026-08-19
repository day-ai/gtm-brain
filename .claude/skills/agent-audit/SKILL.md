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
   triggered vs manual, right cadence), and effectiveness from `manage_skills → get_history` (firing recently?
   output substantive vs hollow? delivered — per the run's `notification.delivered`, not channel config?). And
   identity quality: name, title, and especially the description (the agent's system prompt). The one
   thing you still can't see is engagement depth (does a human act on the output?) — name it plainly
   as unmeasured.
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
| Skills well-written AND properly automated | {N}/{N} |
| Scheduled skills confirmed firing with substantive, delivered output | {N}/{N} |
| People who should be in the workspace but aren't | {N} |
| Recommended new agents · seats they'd need · ~free moves (no new seat) | {N} · {N} · {N} |

**Where this team is:** {one line — e.g. "using Day AI as a shared contact list, not as GTM automation."}
**The single highest-leverage move:** {one line}
**The cost to close the gap:** {one line — e.g. "3 new seats for the 3 active sellers; the other 5 highest-value moves need no new seat."}
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
- `/start`                           → see how this moves your initiatives (agent coverage + delivery are core `bootstrap-day-ai` criteria)
- Send the draft nudge emails above from your own inbox to the people who should be here
```

---

## Notes

- **Recommend-only. No invites, no edits, no skill writes.** This skill sees clearly and proposes; `/implement` acts.
- **The draft nudge emails are a first-class output** — they're often the single highest-ROI thing in the report, because re-engaging an already-invited person is nearly free. Draft them in the operator's voice, specific to each person, short and human.
- **Confirm effectiveness from `manage_skills → get_history`, not config.** Read recent runs: firing? substantive vs hollow output? delivered (per the run's `notification.delivered` boolean — `slackFailureReason` explains a failure, and a `null` `notification` means nothing was configured to send: a config gap, not a failed delivery — never the channel config)? Firing + substantive + delivered = delivering value. The only thing still unmeasurable is engagement *depth* (does the person act on it?) — a signal the MCP doesn't expose yet; name it as unmeasured. Don't claim "delivering value" from a schedule existing.
- Prioritize by **value per cost**, not raw leverage: a near-free move (re-engaging an already-invited person, rewriting an existing skill, scheduling a skill that already exists) outranks one that needs a new seat unless the value is overwhelming. Standing up Coach + CRM Data Nerd agents for a team of active sellers is high value *and* worth the seats — but say what it costs. Every recommended new agent must carry its value-vs-cost case (job slice + output vs. seat/tier), and the report must total the cost to close the gap so the operator sees the bill before `/implement`, never after. Cost figures follow CLAUDE.md's pricing rules: live source (workspace billing, or https://day.ai/pricing), full math shown, unverifiable figures flagged.
- **Recommending against a Day AI skill is a valid recommendation.** Where a capability gap is best served by the team's own Claude repo — interactive, orchestrating, or operator-only work that a single Day AI prompt would flatten — say so plainly, with the reasoning, instead of proposing a skill that would fire and disappoint. Use the same disposition thinking as `/discover` Gate 5's triage menu.
- Re-run periodically — the maturity table is most useful as a trend.
- **This audit is how `/start` verifies an initiative's agent-coverage and skill-delivery criteria.** When invoked as a grounding step for an initiative (e.g. `bootstrap-day-ai`), the maturity table maps directly onto those success criteria — frame the headline around what the initiative still needs to hit, not just abstract maturity.
