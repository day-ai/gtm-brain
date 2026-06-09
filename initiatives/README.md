# Initiatives

An **initiative** is a bounded, owned, time-boxed effort the GTM team is driving toward — with a clear definition of success you can actually verify. It sits between the standing intent of the planning layer and the fine-grained outcomes that realize it:

```
planning/COMPANY_PLAN.md · planning/STRATEGY.md   ← standing intent (what the business is trying to do)
            │
      initiatives/<slug>.md                        ← bounded, owned, status-tracked efforts toward that intent  ← YOU ARE HERE
            │
   planning/OUTCOMES.md · use cases · agents · skills  ← the fine-grained things an initiative is realized through
```

An outcome is atomic — "draft a follow-up after a call." An initiative is the larger effort an outcome serves — "get the SE team running on Day AI by end of Q3." One initiative is realized through many outcomes, agent builds, invites, and skills. If you can finish it in one `/implement` run, it's probably an outcome, not an initiative.

`/start` is the entrypoint that reads this folder, takes stock of every initiative, and kicks off the work each one needs based on its current state.

---

## File format

One Markdown file per initiative, flat in this folder, named `<slug>.md` (kebab-case, matching the `id`). Each file is **frontmatter + body**.

### Frontmatter (required unless noted)

```yaml
---
id: bootstrap-day-ai                 # stable slug; matches the filename
title: Get Day AI set up and delivering value
status: NEW                          # NEW | IN_PROGRESS | PAUSED | CANCELLED | SUCCEEDED
created: 2026-06-09                  # date the initiative started (absolute, YYYY-MM-DD)
target: 2026-07-21                   # date success should be achieved by (absolute)
creator: christopher@day.ai          # who created the initiative
dri: christopher@day.ai              # directly-responsible individual — optional; omit or leave blank if none
success_criteria:                    # VERIFIABLE statements; "done" is unambiguous and checkable
  - Every active seller has a Coach and a CRM Data Nerd agent, each with a strong identity.
  - At least one scheduled skill per active person is confirmed firing with substantive, delivered output.
sources:                             # where this initiative came from — cite the origin
  - type: meeting                    # meeting | page | doc | conversation | manual
    ref: <Day AI meetingrecording id or share URL>
    note: GTM planning meeting, 2026-06-05
  - type: page
    ref: <Day AI Page id or URL>
    note: Q3 GTM strategy page
---
```

**Field notes**

- **`status`** — exactly one of `NEW`, `IN_PROGRESS`, `PAUSED`, `CANCELLED`, `SUCCEEDED`. `/start` advances it; you can also edit it by hand. See the lifecycle below.
- **`created` / `target`** — the timeframe: when it started and when success is due. Always absolute dates (`YYYY-MM-DD`), never "next month."
- **`creator`** — required. **`dri`** — optional; the single person accountable for the outcome. If there's no DRI yet, omit the key or leave it blank — `/start` will flag it.
- **`success_criteria`** — the heart of an initiative. Each must be **verifiable**: a person (or an agent reading the workspace graph) can look and say "true" or "false" without judgment calls. Prefer criteria that map to something queryable in Day AI (counts of agents, skills confirmed firing via `get_skill_history`, pipeline coverage, a Page existing) over vibes.
- **`sources`** — cite the origin so the initiative is traceable: the recorded planning meeting, the strategy Page, the Slack thread, the conversation. Use Day AI object ids / share URLs where the source lives in the workspace.

### Body

The body **describes the initiative and carries all the relevant context — or pointers to it.** Recommended sections:

- **Why now** — the problem or opportunity, in plain language. The case for spending the team's time and the workspace's cost on this.
- **Context** — everything needed to act, or pointers to where it lives. Link Day AI objects (Pages, meetings, opportunities, contacts) by id/URL rather than copying them; this file should stay a durable index, not a stale snapshot.
- **What success looks like** — prose around the `success_criteria`, and how each will be checked.
- **Plan of attack** — the agents, skills, invites, and outcomes this initiative is realized through, and roughly the order. This is what `/start` reads to know what to kick off.
- **Cost** — what pursuing it consumes (seats, tier budget, people's time), so the value-vs-cost case is explicit. See the cost principle in [`../CLAUDE.md`](../CLAUDE.md).
- **Log** — append-only notes as the initiative progresses (date-stamped). Useful for `/start` to see where things stand.

Copy [`TEMPLATE.md`](./TEMPLATE.md) to start a new one.

---

## Status lifecycle

```
NEW ─▶ IN_PROGRESS ─▶ SUCCEEDED
 │          │
 │          ├─▶ PAUSED ─▶ IN_PROGRESS
 │          │
 └──────────┴─▶ CANCELLED
```

- **NEW** — created, not yet started. `/start` will offer to kick it off.
- **IN_PROGRESS** — actively being worked. `/start` reports progress against `success_criteria` and kicks off the next moves.
- **PAUSED** — intentionally on hold. `/start` lists it but doesn't act unless asked. Note why in the log.
- **CANCELLED** — abandoned. Kept for the record; `/start` ignores it for action.
- **SUCCEEDED** — every `success_criterion` verified true. `/start` confirms the verification rather than taking it on faith ("confirm states from outcomes, not config fields").

**`/start` never silently changes status.** Advancing to `SUCCEEDED` requires the criteria actually verifying against the workspace; `CANCELLED`/`PAUSED` are operator decisions. The skill proposes; the operator confirms.

---

## The default initiative

[`bootstrap-day-ai.md`](./bootstrap-day-ai.md) ships with this repo. It's the first initiative every workspace runs: get Day AI connected, the people in, each person running the agents they should, and the planning layer real and synced. It replaces what `/setup` used to do — the same first-run work, now framed as the initiative it always was, with verifiable success criteria and a status you can track. `/start` runs it first.
