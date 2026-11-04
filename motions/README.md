# Motions

A **motion** is a repeatable GTM activity your team runs over and over: discovery, outbound messaging, demos, pilots, scoping, renewals. This folder registers the motions you've made **self-improving**: each one has a written way to run it, an evaluator that studies how the team actually runs it against what happens to the deal afterward, and a path for what works to reach every rep's agent.

```
        ┌─────────────┐
        │ 1. EXECUTE  │  reps + agents run the motion from the current guide and managed skill
        └──────┬──────┘
               ▼
        ┌─────────────┐
        │ 2. CAPTURE  │  every call, email, and meeting lands in customer memory
        └──────┬──────┘
               ▼
        ┌─────────────┐
        │ 3. ANALYZE  │  the evaluator compares execution against outcomes, across the team
        └──────┬──────┘
               ▼
        ┌─────────────┐
        │ 4. IMPROVE  │  the guide is edited, automatically (bounded) or with the owner's review
        └──────┬──────┘
               ▼
        ┌─────────────┐
        │5. DISTRIBUTE│  managed skill + routing put the current version in every agent's hands
        └──────┬──────┘
               └──────────► back to EXECUTE
```

**Analysis is the engine; customer memory is the fuel.** Without continuous analysis of real interactions there is no loop, just a document going stale. And the analysis only works when every rep's calls sit in one shared, permissioned memory linked to the deals they belong to.

## Motions vs. initiatives

An initiative (`initiatives/`) is **bounded**: it has a target date and finishes. A motion is **standing**: it runs every week for as long as the team sells. Standing up a motion's loop is often an initiative ("discovery is self-improving by Q1"). The motion file is what remains once that initiative succeeds.

## The five parts in Day AI

| Stage | Lives in | Notes |
|-------|----------|-------|
| Execute | A **guide Page** (workspace-shared folder) plus a **managed workspace-library skill** (`deploymentMode: "SHARED"`) deployed to every seller's agent | The skill reads the page at run time and never embeds it. **Content lives in the page; behavior lives in the skill.** |
| Capture | Meeting recordings, email, Slack in customer memory | Set by `/discover` Gate 1 (call-capture posture). With no capture there is no loop. Say so and stop. |
| Analyze | An **evaluator skill** on the motion owner's agent (weekly), plus `/flywheel run` from this repo for deep or bulk passes | Compares cohorts defined by outcomes, not calls against the guide. It sees what its host agent can see, so `/flywheel design` checks that the host's visibility covers the team. |
| Improve | `update_page` on the guide, with a changelog section on the page and in the motion file | Two modes, below. |
| Distribute | The managed skill (one version for everyone) plus a **routing line** in the workspace instruction | Routing means reps don't have to know the skill exists: they ask their agent to prep a call and the current guide shows up. |

## The two modes

**Human in the loop (`mode: human_in_loop`, the default).** The evaluator proposes; the motion's `owner` (an enablement lead, sales manager, or the senior rep who owns craft) accepts, edits, or rejects each recommendation, and adds judgment the data can't show yet (a repositioning, a new segment). Choose this for regulated industries, teams mid-repositioning, or orgs where enablement owns the playbook.

**Automatic, bounded (`mode: automatic`).** The in-workspace evaluator applies **guide-page edits in the categories listed under `auto_apply`**, without waiting for review, and proposes everything else. This depends on the evaluator's host agent being able to edit pages during a skill run; `/flywheel design` verifies that. If it can't, automatic mode runs through `/flywheel run`, where the eligible edits are applied as one pre-approved batch with a single operator yes. Choose this when the motion is well defined and the outcome signal is clear. The bounds hold in every case:

- **Page content only.** Auto-apply never touches a skill prompt, the routing line, the workspace instruction, an agent identity, or anything listed in `never_auto`. Those always go through `/implement`'s preview gate.
- **Declared categories only.** If an edit isn't in `auto_apply`, it's a proposal. Categories are narrow and additive by design ("add a question to the bank," "reword a question," "add an example of a strong answer"); removals and framework changes are not auto-apply categories.
- **Evidence threshold.** An auto edit needs at least `min_evidence` cited calls across at least two reps. Below that it becomes a proposal. A proposal needs at least two cited calls; below that, the evaluator notes the pattern as one to watch and proposes nothing.
- **A cap per run.** At most `max_auto_edits` edits per evaluator run; anything beyond waits for the owner.
- **Logged and reversible.** Every auto edit appends a dated changelog entry to the guide page (what changed, the mode, and the call ids; never rep names or rankings, since the page is workspace-visible) and notifies the owner in the same delivery as the run's proposals. The owner can revert any entry; a reverted category is a signal to narrow `auto_apply`.

*Automatic optimizes for speed. Human in the loop optimizes for judgment. Both run on the same analysis.*

Changing a motion's mode or its `auto_apply` list is itself a reviewed change: the evaluator skill carries the categories in its prompt, so the change ships through `/implement` like any other skill update.

## File format

One file per motion, `motions/<id>.md`, frontmatter plus body. Copy [`TEMPLATE.md`](./TEMPLATE.md). [`discovery.md`](./discovery.md) ships as the worked example because discovery is the easiest motion to evaluate: every rep runs it, the calls are recorded, and what happens to the deal afterward is visible.

**Frontmatter fields**

- **`owner`**: the human accountable for the guide and the reviewer in human-in-the-loop mode. Must appear in `workspace/PEOPLE.md`.
- **`guide_page`** / **`managed_skill`** / **`evaluator_skill`**: Day AI object ids. Blank until `/flywheel design` deploys them.
- **`routing`**: `workspace_instruction` (every agent) or `agent:<ids>` (only the named agents, for a segment variant).
- **`outcome_signal`**: what "it worked" means for this motion, phrased so it can be read from the graph (stage progression, days in stage, closed-won). Never a feeling.
- **`cohorts`**: the comparisons the evaluator makes. Each compares a group defined by outcomes against everyone else.
- **`lenses`**: what the evaluator looks at inside each call.
- **`cadence`**: `weekly` (the platform's schedule shapes are daily, weekdays, weekly, every 4 hours).
- **`mode`**, **`auto_apply`**, **`never_auto`**, **`min_evidence`**, **`max_auto_edits`**: the improvement mode and its bounds, above.
- **`variants`**: optional segment variants (enterprise vs. mid-market, new logo vs. expansion), each its own page section or page, routed by agent.

**Body:** why this motion, the evaluator prompt (grounded in the workspace's real stages and segments), the review protocol, and an append-only **Changelog** of every shipped edit with its evidence.

## Who reads this folder

- `/flywheel` designs, runs, and personalizes motions.
- `/agent-audit` and `/brain-health` check each motion's loop is actually turning (evaluator firing, guide changing, edits cited).
- `/discover` nominates the first motion and its owner at Gates 2 and 5.
- `/start` lists motions under the initiative board, so a stalled loop is visible every session.

## Beyond discovery

The same pattern applies to any motion with a measurable outcome. Messaging is the second-easiest because replies and meetings come back fast.

| Motion | What the guide covers | What the evaluator looks at | Outcome signal |
|--------|-----------------------|-----------------------------|----------------|
| Discovery | Questions, qualification framework, call prep | Questions top reps ask, coverage gaps | Stage progression, cycle length, win rate |
| Messaging / outbound | Positioning, value props, email structure | Which messages, angles, and personas get replies | Reply rate, meetings booked, pipeline created |
| Pilots / POCs | Success criteria, structure, check-ins | Pilot designs that convert vs. stall | Pilot-to-close rate, time to decision |
| Scoping | Requirements gathering, pricing inputs | Scoping that leads to a clean close vs. rework | Slippage, discounting, redlines |
| Demos | Narrative, use cases by persona | Which demo flows advance deals | Next-step rate, stakeholder expansion |
| Renewals / expansion | Health signals, expansion plays | Which plays drive expansion | Net retention, expansion pipeline |

Start with one motion, prove the loop turns, then add the next.
