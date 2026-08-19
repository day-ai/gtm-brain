---
id: map-your-gtm
title: Map your GTM before you connect
status: NEW
created: 2026-08-13
target:                # set when this initiative kicks off
creator:               # /start fills this with the confirmed operator
dri:
success_criteria:
  - workspace/TECH_STACK.md is complete, every system a customer touches is named, each has an owner, and the migration posture (CRM Migration vs Customer Memory) is decided and signed off.
  - planning/COMPANY_PLAN.md, planning/STRATEGY.md, and planning/OUTCOMES.md are non-empty and coherent, the top 1-3 business priorities are stated, and every domain owner / SME is named in workspace/PEOPLE.md with what they own.
  - workspace/PEOPLE.md holds the full roster with intended Day AI roles, a named rollout lead and exec sponsor, a per-seat activation owner for every user, and the operator of this repo recorded with a trust note.
  - workspace/PRIVACY.md explains how Day AI's per-user privacy settings work and records the operator's answer to the recommended-setup question: either a recommended setup per persona (email sharing, recording mode, meeting sharing, exclusion intent) with open questions named, or an explicit decision to leave settings to each user.
  - initiatives/first-win.md exists with numeric success criteria (baseline, target, judge, by-date) and a scoped user set of no more than ~5 people.
  - rollouts/preflight/ holds the full payload (workspace instruction, custom properties, pages, invites, agent specs with creation cards, skills, import plan, enablement assets) with the first-win slice marked PRIORITY, and every skill's deferred grounding is marked GROUND-AFTER-CONNECT.
sources:
  - type: manual
    ref: initiatives/README.md
    note: Ships with the GTM Brain repo as the pre-signup default initiative. Leads on any clone without a connected workspace.
---

# Map your GTM before you connect

## Why now
You don't need a Day AI workspace to build your GTM Brain. Everything a workspace needs at launch, except three manual steps, can be authored here as files and applied through the MCP the day you connect. Doing the mapping now, while you're evaluating, means signup day is an apply step: your agents deliver output grounded in your business in their first session, your team arrives to role-specific guides instead of a blank screen, and you (the rollout lead) arrive on launch day as the person who designed the fleet.

Treat the harness the way a good founder treats a paid consultant: give it everything, answer its questions honestly, and listen to what it says. It will do the same in return, and it will tell you plainly when it can't verify something yet.

## Context
- **Operating principles**: [`../CLAUDE.md`](../CLAUDE.md). The pre-signup operator state is described there.
- **The discovery engine**: `/discover` runs the interviews and reads your existing materials. Drop anything useful in `discovery/inbox/` (your GTM docs, CRM exports, org chart, an existing Claude Project's instructions, repos with CLAUDE.md or `.claude/skills/`). The more it reads, the less it asks.
- **What the brain needs to know and why**: [`../docs/INSTRUCTION_ARCHITECTURE.md`](../docs/INSTRUCTION_ARCHITECTURE.md) explains where every rule you write will live in Day AI.
- **Where output lands**: the planning docs, `workspace/TECH_STACK.md`, `workspace/PRIVACY.md`, `workspace/PEOPLE.md`, and the deployable payload in `rollouts/preflight/` (see its README).
- **This repo should be private** before anything real goes in it. It will hold your strategy and candid notes. See the README's "Your team's GTM Brain repo."

## What success looks like
The six criteria above, verified from the repo itself. Pre-signup, verification means decisions, not files existing: a PRIVACY.md that never asked the recommended-setup question is not done, and a fleet design with no cost case is not done. When a workspace later connects, `/start` reconciles everything mapped here against the live workspace and the remaining criteria verify the normal way.

## Plan of attack
`/discover` drives all of this; `/start` reports progress each session. The first-win track runs ahead of the full map on purpose.

1. **Outcome interview + first win.** `/discover` starts with what the business is trying to achieve, then proposes 1-3 workflows your current stack cannot do at all, ranked by three filters: provable in a short window, not integration-gated, and reading data that is ready. Pick one, scope it to yourself or up to ~5 users, and `/discover` writes `initiatives/first-win.md` with numeric, dated success criteria. That slice of the payload deploys first at connect time.
2. **Gate 1: Tech stack + customer memory.** Every system a customer touches, its owner, and the migration posture. Ingests whatever is in `discovery/inbox/` first. *(criterion 1)*
3. **Gate 2: Strategy + levers + domain owners.** The three-layer plan, the top 1-3 priorities, and the SMEs who will own the living guides. Existing strategy docs and playbooks get a per-doc decision here: convert to a workspace Page, or stay in their source system. *(criterion 2)*
4. **Gate 3: People + permissions.** Roster, intended roles, rollout lead, exec sponsor, and a named activation owner per seat. *(criterion 3)*
5. **Gate 4: Privacy.** Educational, not contractual: every user applies their own settings when they activate. `/discover` explains how the settings work, asks whether you want to advise a recommended setup across your users, and records the answer per persona with open questions named. Fourth in conversation order because guidance is per-persona, so the roster must exist first. *(criterion 4)*
6. **Gate 5: Fleet design.** Archetypes mapped to roles, identities written, tiers and slot budgets assigned, delegation wired, existing skills adapted rather than rewritten, and a value-vs-cost case per agent (costs flagged as estimates until confirmed with Day AI). Uses `/design-agent`'s spec format.
7. **Build the payload.** Assemble `rollouts/preflight/` and the enablement assets. *(criteria 5-6)*
8. **Ready-to-connect checklist.** Privacy guidance captured, roster defined with activation owners, sources owned, agent tiers locked, success criteria numeric and dated. Then sign up, connect the MCP, rerun `/start`, and `/implement` applies the payload with previews at every step, reviewing the privacy guidance before invites go out.

## Cost
Pre-signup, the cost is your time and your colleagues' interview time. The payload carries the future bill explicitly: seats per person in INVITES.md and tier costs per agent in each spec, priced per CLAUDE.md's pricing rules (current figures from https://day.ai/pricing, the full seats + tiers + slots math shown, anything the page can't answer flagged "confirm with Day AI"). Nothing in this initiative spends money; it makes the future spend legible before you commit to it.

## Log
<!-- append-only, date-stamped, newest at the bottom -->
- 2026-08-13: created as the pre-signup default initiative.
