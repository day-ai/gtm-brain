---
id: <slug-matching-filename>
title: <one-line name of the initiative>
status: NEW                       # NEW | IN_PROGRESS | PAUSED | CANCELLED | SUCCEEDED
created: <YYYY-MM-DD>             # when it started
target: <YYYY-MM-DD>             # when success is due
creator: <email>                 # who created it
dri: <email>                     # directly-responsible individual — optional; omit if none yet
success_criteria:                # each must be VERIFIABLE — true/false without judgment
  - <criterion 1>
  - <criterion 2>
sources:                         # where this initiative came from
  - type: meeting                # meeting | page | doc | conversation | manual
    ref: <Day AI object id / share URL>
    note: <what it is>
---

# <title>

## Why now
<The problem or opportunity in plain language. The case for the team's time and the workspace's cost.>

## Context
<Everything needed to act — or pointers to where it lives. Link Day AI objects (Pages, meetings,
opportunities, contacts) by id/URL rather than pasting them in, so this file stays a durable index.>

## What success looks like
<Prose around the success_criteria above, and how each will be checked — ideally something queryable
in the workspace graph.>

## Plan of attack
<The agents, skills, invites, and outcomes this initiative is realized through, roughly in order.
This is what /start reads to know what to kick off. Reference the skills by name: /plan, /agent-audit,
/design-agent, /audit, /implement, /sync-pages.>

## Cost
<What pursuing this consumes — seats, tier budget, people's time. Make the value-vs-cost case explicit.>

## Log
<!-- append-only, date-stamped, newest at the bottom -->
- <YYYY-MM-DD> — created.
