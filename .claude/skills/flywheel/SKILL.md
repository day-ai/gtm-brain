---
name: flywheel
description: >-
  Make a GTM motion self-improving. `design` stands up the loop in the
  workspace (guide page, managed skill, routing line, evaluator skill, and
  the motions/ file); `run` is the bulk evaluator from Claude Code: it
  compares how the motion is executed against deal outcomes across the team
  and proposes guide and skill edits with cited calls; `personalize` finds
  each rep's gaps and recommends team variants and coaching overlays.
  Discovery is the default motion. Usage: /flywheel <motion> [design | run |
  personalize]
metadata:
  internal: true
---

# /flywheel

A playbook goes stale the day it's written. This skill closes the loop around one motion: **execute → capture → analyze → improve → distribute**, then repeat. The concepts, the two modes, and the file format live in `motions/README.md`; read it first.

$ARGUMENTS

The first argument is the motion (`discovery` if omitted). The second is the mode of this run:

- **No `motions/<motion>.md`, or its ids are blank** → `design`.
- **Loop deployed** → `run`, unless the operator asked for `personalize`.

Confirm the choice in one line before starting.

---

## Preconditions (every mode)

1. **Operator state.** Follow `/start`'s detection. `design` and any write need a connected Owner/Admin. `run` and `personalize` read the whole team's calls and need a connected Owner/Admin too: a Member sees only what they can see, which makes a team comparison wrong. Pre-signup, `design` drafts into the preflight payload (see Design, step 6) and the other modes stop with that reason.
   **Pre-signup `design` skips preconditions 2 and 3**: there is no graph to check yet. Record each as a `GROUND-AFTER-CONNECT` gate in the drafted evaluator skill (confirm capture coverage per rep; confirm stage names and outcome readability), so the evaluator doesn't go live until both pass.
2. **Capture exists.** Read the call-capture posture in `workspace/TECH_STACK.md` and confirm with `search_objects` that recordings for this motion actually exist in the last 30 days, across more than one rep. If calls aren't captured for most of the team, the analysis would be wrong: say so, name the fix (the capture posture, per-user recording modes), and stop. A loop on half the calls teaches the wrong lessons.
3. **Outcomes are readable.** Confirm the `outcome_signal` can be read from the graph: `read_crm_schema` for the real pipeline and stage names, `search_objects` on opportunities for current stage, owner, and created/close dates. Check whether stage-change history (when each deal entered each stage) is available from the connected tools. If it isn't, fall back to current stage plus created date, and say in the report that cycle-length cohorts are approximate. Never fabricate a stage or a property.

---

## Design: stand the loop up

Spawn the **`data-analyst`** for grounding and the **`agent-implementor`** to draft. Nothing writes until `/implement`'s preview is approved.

1. **The owner.** Who owns this motion's craft? Read `workspace/PEOPLE.md` and `/discover` Gate 2's domain owners. If nobody is named, ask. A loop with no owner is a document going stale with extra steps.
2. **The guide page.** Find an existing guide (`search_objects`, `read_page`) or an approved source doc (`/discover` Gate 2's doc-to-Page list). Draft or freshen it with the sections in `motions/discovery.md` (adapted to the motion) and a **Changelog** section at the bottom. File it in a folder created with `shareWithWorkspace: true`.
3. **The managed consumer skill.** One workspace-library skill (`deploymentMode: "SHARED"`), written to the `/write-skill` bar, that does the motion's agent-side work: for discovery, prep before each first call and a post-call summary against the guide. It names the guide page by title and objectId and **reads it at run time**; it never embeds the content. Deploy it to every seller who runs the motion (`manage_skills → deploy`), respecting each target agent's automated-skill slots. One version for everyone is the point. Where a seller's Coach already has a meeting-prep skill, propose retiring or narrowing it in the same change set (e.g. to non-discovery meetings), so the seller gets one prep, not two.
4. **Routing.** Draft one line for the workspace instruction: *"When anyone asks for help with discovery, call prep, or qualification, follow the Discovery Guide page ({objectId})."* Merge it into the live text (`list_configuration` first, 3000-character cap, full current → proposed text in the preview). For a segment variant, route at the agent layer instead: add the line to the identity of only the agents that should use it.
5. **The evaluator skill.** A weekly SCHEDULE skill on the **owner's** agent (one automated slot on their tier). **Check the owner's visibility first.** The evaluator sees what the owner's agent can see. Compare the per-rep count of the motion's recordings visible to the owner (`search_objects`, scoped to them) with the team-wide count, and read the sharing guidance in `workspace/PRIVACY.md`. If the owner can't see most reps' calls, the comparison will be skewed: host the evaluator on an Owner/Admin agent with full visibility and deliver to the motion owner, or fix sharing first. Either way, the evaluator reports its coverage (calls and reps it could read) on every run. Its prompt is the motion file's evaluator prompt, grounded with real stage names, segments, and the page id. It also carries the improvement mode:
   - **human_in_loop**: it proposes; it never edits the page. It delivers proposals to the owner (email for the full analysis, a Slack DM pointing at it).
   - **automatic**: first verify that the hosting agent can edit pages during a skill run. Check its tier and its tools; this harness documents `update_page` as an MCP tool, not as a guaranteed in-skill capability. If it can't, the evaluator proposes like human_in_loop and automatic mode runs through `/flywheel run` only. If it can, it may apply `update_page` edits **only** in the `auto_apply` categories, **only** with `min_evidence` cited calls across at least two reps, **at most** `max_auto_edits` per run, each with a changelog entry on the page. It never removes content and never touches anything in `never_auto`. It reports auto edits and remaining proposals to the owner in one delivery.

   Write the categories and bounds into the prompt verbatim. That makes a change of mode a reviewed skill update, never a silent drift.
6. **The motion file.** Fill `motions/<motion>.md` (from `motions/TEMPLATE.md`): owner, cohorts, lenses, mode, bounds, and the ids once deployed. **Pre-signup:** draft the guide into `rollouts/preflight/PAGES/`, the consumer and evaluator skills into `rollouts/preflight/SKILLS/` with `GROUND-AFTER-CONNECT` markers for stage names and page ids, and the routing line into `rollouts/preflight/WORKSPACE_INSTRUCTION.md`. Preflight apply wires them up on connect day.
7. **Hand off to `/implement`** with the change set in dependency order: folder → page → workspace instruction → consumer skill + deploy → evaluator skill. The preview shows the evaluator's mode and bounds plainly, so the operator approves the governance along with the prompt.

---

## Run: the bulk evaluator, from Claude Code

The in-workspace evaluator handles the weekly pass. Run this for a deeper pass, a first baseline, a structural change (new section, new variant, framework change), or whenever the operator wants to see the evidence directly.

### 1. Gather
- Read `motions/<motion>.md` and the current guide (`read_page`). Analysis is against the guide as it is today.
- Pull the motion's calls for the window (default: last 30 days for the motion, trailing 90 days for rep-level conversion): `search_objects` for meeting recordings, then `get_meeting_recording_context` per call. Tie each call to its opportunity, its rep, and the opportunity's outcome.
- Work in batches, and use parallel subagents for large windows (give each a slice of calls and the lenses; collect structured notes back). Record the counts: calls read, reps covered, deals covered, calls excluded and why.

### 2. Compare cohorts, not calls against the guide
For each cohort in the motion file, compare the cohort's calls against everyone else's on each lens. The question is always: **what does the group with better outcomes do differently, and does the guide ask for it?**

- Approaches over-represented in the winning cohort → candidate additions or emphasis.
- Guide content the winning cohort skips without penalty → candidate for review (never auto-removed).
- Guide content most reps skip and the losing cohort skips more → a coverage gap to reinforce.
- Patterns that differ by segment, industry, or persona → a candidate variant.

Be honest about strength. Small cohorts, one rep dominating a pattern, or a confounder (a segment that's easier to win) get named. Correlation is not proof; the owner's judgment is the check.

### 3. Write the report
Save to `rollouts/<YYYY-MM-DD>-<motion>-eval/REPORT.md`:

```markdown
# {Motion} evaluation — {date}

**Window:** {dates} · **Calls:** {N} from {N} reps on {N} deals · **Outcome signal:** {…}{, approximate: stage history unavailable}
**Mode:** {human_in_loop | automatic} · **Guide read:** {page title, objectId, last changelog date}

## What the winning cohorts do differently
1. {approach} — {cohort}: {X of Y calls} vs. everyone else: {X of Y} — e.g. {call id, rep, one-line quote}
… (up to 5)

## Gaps between the guide and practice
1. {gap} — {how often, who} — e.g. {call ids}
… (up to 3)

## Proposed edits
| # | Target | Section | Before → after | Evidence (≥3 calls, ≥2 reps) | Auto-eligible? |
|---|--------|---------|----------------|------------------------------|----------------|
| 1 | guide page | Question bank / Decision process | — → "Who else signs off on a purchase like this?" | {ids} | yes: add question, {N} calls / {N} reps |
| 2 | managed skill | prep step | {…} | {ids} | no: skill prompt |

## Variants worth considering
{segment, what differs, evidence, or "none yet"}

## What I couldn't measure
{stage history, reps with sparse capture, cohorts too small}
```

**The output is a proposed change with evidence, not a report.** Every proposed edit names its target, its section, the exact before and after, and the calls behind it (at least two; auto-eligible edits need `min_evidence` calls from at least two reps). If nothing clears the evidence bar, say so; "no change this week" is a valid result.

### 4. Apply
- **Guide edits, human_in_loop:** walk the owner (or the operator, on the owner's behalf) through each edit: accept, edit, or reject. Accepted edits go to `/implement` as targeted `update_page` edits plus changelog entries.
- **Guide edits, automatic:** the harness still previews before it writes (CLAUDE.md). The edits that meet every bound (an `auto_apply` category, the evidence threshold, within the cap) go to `/implement` as one batch marked **pre-approved by the motion's policy**: the operator sees them and applies the batch with a single yes, without reviewing each one. Everything else follows the human_in_loop path. Unreviewed application happens only in the in-workspace evaluator, inside the bounds written into its prompt.
- **Skill, routing, variant, or framework changes:** always through `/implement`'s preview, in either mode. If the change adds a section the consumer skill should act on, update the skill in the same change set so the page and the skill never diverge.
- Append each shipped edit to the motion file's **Changelog**: date, what changed, mode (auto, or reviewed by whom), evidence.
- **What goes on the shared page.** The guide's changelog is workspace-visible: it carries the date, the edit, the mode, and **call ids only**. Rep names, cohort membership ("top 20%"), and quotes attributed to reps stay in the owner's delivery and in this repo's `rollouts/` report. A guide page must never read as a ranking of the team.

---

## Personalize: individual strengths and gaps (recommend-only)

The shared guide is the floor. This mode finds where each rep differs from it and recommends how to close the gap. **It makes no writes in this version**: it produces recommendations that go through `/design-agent` and `/implement` like any other change.

1. Run the cohort analysis per rep: for each rep, coverage on each lens compared with the winning cohort. Name strengths as well as gaps.
2. Recommend, per rep, a **coaching overlay**: a short paragraph in the identity description of that rep's Coach (or their own agent) that targets their gaps ("before first calls, make sure decision process and sign-off are covered; suggest two questions from the guide's Decision process section"). Recommend **team variants** where a segment's winning pattern differs from the shared guide.
3. Apply CLAUDE.md's rule about teammates without exception: an overlay **bakes in the behavior and never the reason.** It contains no assessment of the rep, no comparison with colleagues, no scores. "Cover decision process" belongs in it; "you rarely uncover decision process" does not. The per-rep analysis stays in the operator's private repo under `rollouts/` and never goes into a page or a prompt.
4. Respect tier budgets and the one-version rule: the overlay lives in the rep's agent identity, which shapes every run that agent does (including the managed prep skill), so it takes no automated slot and leaves the managed skill identical for everyone. A team variant is a separate guide section or page with agent-level routing, never a fork of the managed skill.
5. Save to `rollouts/<YYYY-MM-DD>-<motion>-personalize/REPORT.md` with a per-rep table (rep, strengths, gaps, overlay text, evidence) and hand off to `/implement {rep}` for the overlays the operator approves.

---

## Notes

- **One motion at a time.** Prove discovery turns before designing the next. The motion table in `motions/README.md` lists what to do next (messaging is second-easiest).
- **Capture is the precondition, not a detail.** The loop is only as good as the share of the team's calls it can see.
- **Distribution is the point.** An improved guide no agent reads is shelf-ware. Check the consumer skill actually reads the page (`manage_skills → get_history`) before calling the loop live.
- **The evaluator is verified like any skill.** Firing weekly, substantive output, `notification.delivered` true. A loop whose evaluator is dormant is a stalled flywheel; `/brain-health` flags it.
