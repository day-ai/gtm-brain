---
id: discovery
title: First discovery call
owner:                           # TODO: the enablement lead, sales manager, or senior rep who owns discovery craft
guide_page:                      # set by /flywheel design
managed_skill:                   # set by /flywheel design
evaluator_skill:                 # set by /flywheel design
routing: workspace_instruction
outcome_signal: opportunity advanced past the discovery stage within 21 days of the first call   # TODO: confirm stage names via read_crm_schema
cohorts:
  - top 20% of reps by discovery-to-next-stage conversion, trailing 90 days
  - deals that closed won in under 45 days
  - deals that advanced past discovery vs. deals that stalled in it
lenses:
  - questions asked, grouped by theme (pain, impact, current process, decision process, timeline, champion)
  - qualification coverage against the team's framework; which elements are consistently missed
  - question depth and follow-up (does the rep dig into pain or move on?)
  - objections raised and how reps in the winning cohort handled them
  - differences by segment, industry, or persona
  - gaps between the guide and what reps actually do
cadence: weekly
mode: human_in_loop
auto_apply: []                   # if switched to automatic, a sensible start: [add question, reword question, add strong-answer example]
never_auto:
  - qualification framework and required elements
  - red flags and disqualifiers
  - positioning and pricing language
  - removing a question or section
min_evidence: 3
max_auto_edits: 3
variants: []                     # e.g. [enterprise, mid-market] once the evaluator shows they differ
---

# First discovery call

## Why this motion
Every seller runs discovery, the calls are recorded, and you can see what happens to each deal afterward. That makes it the easiest motion to evaluate and the clearest place to prove the loop before adding messaging, demos, or pilots.

## The guide
A workspace-shared **Discovery Guide** page. Suggested sections for v1 (skip any that don't fit how this team sells):

- **Objectives**: qualify, uncover pain, map the buying process.
- **Qualification framework**: MEDDPICC, BANT, or the team's own, and which elements a first call must cover.
- **Question bank**, grouped by theme: pain, impact, current process, decision process, timeline, champion.
- **Call prep checklist**: what to research about the account and attendees beforehand.
- **Red flags and disqualifiers.**
- **What good looks like**: a strong first call vs. a weak one.
- **Post-call outputs**: summary, next steps, record fields to update.
- **Changelog**: maintained by the evaluator; every shipped edit with its date, mode, and cited calls.

If the team already has a discovery doc, `/discover` Gate 2 converts it (with per-doc approval) rather than writing a new one.

## Evaluator prompt
Grounded at `/flywheel design` time with real stage names, segments, and the page id:

> Review every first discovery call from the last 30 days. Compare the questions asked by the top 20% of reps by discovery-to-next-stage conversion against everyone else, and look separately at deals that closed won in under 45 days. Read the current Discovery Guide first. Identify the 5 questions or approaches most associated with progression, and the 3 most common gaps between what reps do and what the guide asks. For each, propose a specific edit to the guide (the section, the before, the after) and cite the calls that justify it: at least two calls per proposal, and at least {min_evidence} from at least two reps for any edit applied automatically. If the evidence is thin, say so and propose nothing. Report how many calls and reps you could read.

## Review protocol
Weekly. The evaluator delivers its proposals to the owner (email for the full analysis, a short Slack DM pointing at it). The owner accepts, edits, or rejects each; accepted edits ship to the guide page with a changelog entry that cites call ids only (rep-level detail stays in the owner's delivery). Structural changes (a new section, a segment variant, a framework change) go through `/flywheel run` → `/implement` so the managed skill and routing stay in step with the page.

## Changelog
<!-- append-only, newest at the bottom: date — what changed — mode (auto | reviewed by <name>) — evidence (call ids) -->
- 2026-11-04 — shipped as the worked example; not yet deployed.
