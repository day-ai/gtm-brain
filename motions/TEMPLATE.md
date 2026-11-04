---
id: <slug-matching-filename>
title: <the motion, e.g. "First discovery call">
owner: <email>                   # accountable for the guide; the reviewer in human_in_loop mode
guide_page:                      # Day AI page id — set by /flywheel design
managed_skill:                   # workspace-library skill id (deploymentMode SHARED) — set by /flywheel design
evaluator_skill:                 # skill id on the owner's agent — set by /flywheel design
routing: workspace_instruction   # workspace_instruction | agent:<id>,<id>
outcome_signal: <readable from the graph, e.g. "advanced to stage 2 within 21 days">
cohorts:                         # each: an outcome-defined group vs. everyone else
  - <e.g. top 20% of reps by stage-2 conversion, trailing 90 days>
lenses:                          # what the evaluator examines inside each call
  - <e.g. questions asked, qualification coverage>
cadence: weekly
mode: human_in_loop              # human_in_loop | automatic
auto_apply: []                   # automatic mode only: narrow, additive page-edit categories
never_auto:                      # always proposals, in either mode
  - <e.g. qualification framework>
min_evidence: 3                  # cited calls, across ≥2 reps, required for an auto edit
max_auto_edits: 3                # per evaluator run
variants: []                     # optional segment variants, each routed by agent
---

# <title>

## Why this motion
<Why it's worth a loop: how often it runs, how clear the outcome signal is, what a better version is worth.>

## The guide
<What the guide page covers, its sections, and where it lives. Link the page; don't paste it.>

## Evaluator prompt
<The analysis the evaluator runs, grounded in the workspace's real stage names and segments. Output is a
proposed edit with evidence, never just a report.>

## Review protocol
<Who reviews, when, and where proposals land (a page, a Slack DM, an email). In automatic mode, how the owner
hears about auto edits and how they revert one.>

## Changelog
<!-- append-only, newest at the bottom: date — what changed — mode (auto | reviewed by <name>) — evidence (call ids) -->
- <YYYY-MM-DD> — created.
