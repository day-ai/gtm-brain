---
id: bootstrap-day-ai
title: Get Day AI set up and delivering value
status: NEW
created: 2026-06-09
target: 2026-07-21
creator: christopher@day.ai
dri:
success_criteria:
  - The team's own private GitHub repo exists (made from this template), with the operators added as collaborators, and they can pull/push the plan, initiatives, and PEOPLE.md.
  - The Day AI MCP server is connected and the operator's role is confirmed (Owner/Admin for the full harness).
  - workspace/PEOPLE.md reflects the real roster with roles, and no key player from the CRM is left unplaced or uninvited.
  - planning/COMPANY_PLAN.md, planning/STRATEGY.md, and planning/OUTCOMES.md are non-empty, coherent, and free of unresolved blocking TODOs.
  - Every active person is running at least two agents that fit their role; every active seller has a Coach and a CRM Data Nerd, each with a strong (non-generic) identity.
  - At least one scheduled skill per active person is confirmed firing with substantive, delivered output (verified via `manage_skills → get_history`, not the schedule existing).
  - The planning layer is synced to Day AI Pages so the rest of the company sees the same source of truth.
sources:
  - type: manual
    ref: initiatives/README.md
    note: Ships with the GTM Brain repo as the default first-run initiative; replaces the old /setup procedure.
---

# Get Day AI set up and delivering value

## Why now
This is the first thing any workspace does with GTM Brain. Until the connection is live, the people are in, the plan is real, and each person is running the agents that do their actual job, the workspace is a better contact list — not GTM automation. The point of Day AI is delegated job functions producing work product proactively; this initiative gets a workspace from zero to that bar, and gives you a status you can track instead of a one-time setup you hope stuck.

## Context
Everything needed to run this lives in the repo and the workspace itself:

- **Operating principles** — [`../CLAUDE.md`](../CLAUDE.md). Read it first; it governs every agent here.
- **The shared repo** — this template becomes the team's own **private** GitHub repo, the source of truth the operators sync through. See the README's "Your team's GTM Brain repo" for how to stand it up (Use this template → private → add collaborators).
- **The planning layer** — [`../planning/`](../planning/) (COMPANY_PLAN, STRATEGY, OUTCOMES) and [`../workspace/PEOPLE.md`](../workspace/PEOPLE.md).
- **The agents that do the work** — `gtm-strategist`, `data-analyst`, `agent-implementor` in [`../.claude/agents/`](../.claude/agents/).
- **The agents-are-GTM-automation thesis and the ≥2-agents bar** — see CLAUDE.md and `../.claude/agents/data-analyst.md`. Adding agents costs seats and tier budget, so every agent is justified with a value-vs-cost case (see **Cost** below).
- **Workspace truth** — pulled live through the Day AI MCP (`manage_workspace_members → list_configuration`, `assistant_settings → list`, `manage_skills` incl. `→ get_history`, `search_objects`). Never assume; pull.

## What success looks like
The seven `success_criteria` above, checked against reality, not config:

1. **Shared repo** — a private GitHub repo exists (made from the `day-ai/gtm-brain` template), the operators are collaborators, and they can pull/push. This is the team's source of truth; see the README's "Your team's GTM Brain repo."
2. **Connection + role** — `list_configuration` succeeds and `currentUser.roleName` is known.
3. **People** — the roster in `PEOPLE.md` matches `list_configuration`, and `list_suggested_invites` surfaces no key CRM contact who should be a member but isn't.
4. **Plan** — the three planning docs read coherently and carry no blocking TODOs.
5. **Agent coverage** — `assistant_settings → list` shows every active person with ≥2 role-fit agents; every seller has a Coach and a CRM Data Nerd with real identities (not "Assistant"/blank).
6. **Delivery** — `manage_skills → get_history` shows at least one scheduled skill per active person firing recently with substantive, delivered output. This is the criterion that proves *value*, not just *configuration*.
7. **Synced** — the planning docs exist as Day AI Pages (`/sync-pages`).

## Plan of attack
Run in order; `/start` orchestrates this and reports progress against the criteria each time.

1. **Stand up the shared repo.** Make this template the team's own **private** repo and add the operators as collaborators (README → "Your team's GTM Brain repo"). Confirm they can pull/push. This is the prerequisite that lets operators stay in sync. *(criterion 1)*
2. **Connect + confirm role.** `manage_workspace_members → list_configuration`. Hard gate — if it fails, stop and fix the MCP connection (see the README). Report role and what it allows. *(criterion 2)*
3. **Identify the cast** — spawn `gtm-strategist` to build `workspace/PEOPLE.md` from the roster, agent map, and suggested invites. Confirm placements with the operator. *(criterion 3)*
4. **Scaffold the plan** — `gtm-strategist` does a light first pass at the three planning docs through a short interview. Depth comes later via `/plan`. *(criterion 4)*
5. **`/agent-audit`** — `data-analyst` scores how well the workspace uses Day AI's agents today and hands back a prioritized, costed path: who's missing, who needs which agents, which skills/identities are weak. *(criteria 5–6)*
6. **`/design-agent`** for each gap — design the missing agents (every seller's Coach + CRM Data Nerd first), each with a value-vs-cost case.
7. **`/implement`** — invite the right people, tune identities, deploy skills. Previews before it writes; surfaces the seat/tier bill before deploying.
8. **`/sync-pages`** — publish the plan to Day AI Pages. *(criterion 7)*

## Cost
The setup work itself is free. Closing it is not: criterion 4 means a seat per active person who isn't yet on ≥2 agents, plus the tier budget to support each agent's automated skills. `/agent-audit` and `/implement` total this bill and surface `navigate_to_billing` before anything is deployed — the operator approves the cost, not just the change. An agent added here pays for itself by delegating a real slice of someone's week; one added to move the count is waste.

## Log
<!-- append-only, date-stamped, newest at the bottom -->
- 2026-06-09 — created as the default bootstrap initiative; supersedes the standalone /setup command.
