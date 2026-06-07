# GTM Brain — Operating Guide

This repo is a GTM operating system. A human Owner/Admin and a set of agents work from shared planning documents and execute against a live **Day AI** workspace through the Day AI MCP server.

There are two layers, and they have a strict relationship:

- **The planning layer** (`planning/`, `workspace/PEOPLE.md`) is the source of truth for *what the business is trying to do and who is doing it*.
- **The implementation layer** makes the Day AI workspace *reflect* the plan — the right people, at the right roles, each with an agent configured to do their real work.

Plan first. Implement from the plan. When reality and the plan diverge, the job is to either change the workspace or update the plan — never to let the gap sit silently.

---

## Prerequisites every agent assumes

1. **The Day AI MCP server (`day-ai`) is connected and authorized.** All workspace reads and writes go through it. Identity is implicit in the OAuth token — you never pass `workspaceId`, `userId`, or the current `assistantId`. You only pass `targetAssistantId` when an Owner/Admin acts on a *different* agent than their own.
2. **The operator is an Owner or Admin.** Cross-agent and member-management actions require the `USERS:manage` permission. If a tool returns *"requires Admin or Owner,"* stop and tell the operator — don't design around it. `/setup` confirms the role up front via `manage_workspace_members` → `list_configuration` → `currentUser.roleName`.

---

## The Day AI MCP tools

This harness is built on the workspace-management tools plus the read-only graph tools the Day AI MCP exposes.

### Management tools (the core of this repo)

| Tool | Purpose | Key modes |
|------|---------|-----------|
| `mcp__day-ai__assistant_settings` | Inspect & edit agents — your own, or (Admin/Owner) any agent in the workspace | `read`, `update`, `list` |
| `mcp__day-ai__manage_skills` | Full skill lifecycle on an agent or in the workspace library | `list`, `get`, `create`, `update`, `delete`, `reset_prompt` |
| `mcp__day-ai__manage_workspace_members` | Members, roles, invites, domain auto-invite, suggested invites | `list_configuration`, `invite_member`, `resend_invite`, `revoke_invite`, `update_invite_role`, `enable_auto_invite`, `disable_auto_invite`, `list_suggested_invites`, `navigate_to_billing` |

**Always start a member/invite task with `list_configuration`** — it returns the current members, roles, claimed domains, auto-invite config, and *what the caller is allowed to do*. Read `currentUser` before acting.

**Cross-agent / `list` modes are Admin/Owner-only.** Targeting an agent other than your own (`targetAssistantId`) requires it; so does `assistant_settings` `mode: "list"` and `manage_skills` reading another agent's skills.

**Tier limits are real.** Automated skills (those with a `SCHEDULE` or `EVENT` trigger) consume automated-skill slots governed by the *target agent's* tier. Over-budget creation is rejected with an explanation. When you create a skill for a teammate, the limit checks against *their* tier, not yours. Plan automations around the target agent's packaging; prefer one or two high-value scheduled skills over many.

**Result envelopes** are `{ result: {...} }` on success and `{ error: { message } }` on failure. Permission failures are explicit — surface them, don't retry blindly.

### Graph / read tools (for grounding)

Use the Day AI MCP's read-only graph tools to ground the plan and every skill prompt in what the workspace actually contains — pipeline, contacts, meetings, prior conversations. `search_objects` (general graph search) and `get_meeting_recording_context` (a specific meeting's full context) are the primary ones. Discover the rest from the connected tool list rather than assuming names. **Never fabricate a custom property, pipeline stage, or page that you haven't confirmed exists in the workspace.**

`get_skill_history` (read a skill's recent runs — produced output, firing times, and the `notification.result` delivery) is how the analyst confirms a skill is *actually delivering value*, not just configured. Use the run `notification.result` to confirm delivery — never the channel config. Admin/Owner can read any agent's skill history via `targetAssistantId`.

### Snapshots

There's no separate version history for an agent. The `read`/`list` outputs of `assistant_settings` and `manage_skills` *are* the canonical snapshot of an agent and its skills. If you want to be able to diff or restore, capture those outputs to disk (the implementor does this under `rollouts/`).

---

## Operating principles

**Plan before you touch the workspace.** Build the full picture first: the company plan, the strategy, the outcomes, and who's in the workspace. Acting without context is how you misconfigure a teammate's agent or invite the wrong person at the wrong role.

**Ground everything in real data.** Never write a generic skill prompt or a generic plan. Use the workspace's actual pipeline, meetings, contacts, and the teammate's real role. If swapping one person for another would produce the same skill, it isn't specific enough.

**Do the work, don't just flag it.** When something is wrong — a teammate on a default agent, a skill that's never been tuned, a key contact never invited — fix it (with approval). A report that lists problems without resolving them is overhead.

**Preview, then deploy.** Anything that writes to the live workspace (invites, agent edits, skill creates/updates) is shown to the operator as a plan first and deployed only on approval. The only exception is when the operator explicitly says "deploy."

**Respect the people in the workspace.** A skill you write for a teammate is something *they* will read every day. It must reflect their real work, match their tone, and never contain internal strategy, forecasts, or notes about them. Bake the behavior in without exposing the reasoning behind it.

**Confirm states from outcomes, not config fields.** "This skill is delivering value" is proven by its run output and the teammate engaging with it — not by the existence of a schedule. "This person is active" is proven by recent activity in the graph — not by a seat existing. Don't read a setting as if it were a result.

**The standard is excellence.** Every plan, every skill prompt, every invite should be something you'd be proud to show the whole company. There is no "good enough for now."

---

## Day AI language conventions

Skill prompts and any workspace-facing text the agents produce must use Day AI's language. Re-read drafts against this list:

- Refer to the AI as **"your agent"** or by its name — never "your assistant" or "the bot."
- Refer to the data layer as **"your customer memory"** or **"what I know"** — not "the CRM" or "the database."
- Frame value as **what becomes possible**, not as time saved.
- Name capabilities by what they do: *draft an email*, *search the pipeline*, *send a Slack message*, *update a contact*.
- Structure skills around **what the person needs** ("promises coming due," "relationships going quiet"), not around data sources ("check email, check calendar").
- Don't pad. Give explicit permission to say nothing when there's nothing worth surfacing.

See `.claude/skills/write-skill/SKILL.md` for the full skill-authoring guide. The implementor reads it before writing any prompt.

---

## The data layer

The shared state the team operates on. Agents read from and write to these locations.

| Location | What it is | Written by |
|----------|-----------|-----------|
| `planning/COMPANY_PLAN.md` | Company goals, forecasts, plans (layer 1) | `/plan`, manual |
| `planning/STRATEGY.md` | CRO-level revenue strategy & direction (layer 2) | `/plan`, manual |
| `planning/OUTCOMES.md` | Concrete outcomes that serve layers 1–2 (layer 3) | `/plan`, manual |
| `workspace/PEOPLE.md` | Who's who in the workspace — roles, agents, focus | `/setup`, `/plan` |
| `rollouts/<YYYY-MM-DD>-<slug>/` | Per-run artifacts: audit reports, agent specs, proposed/approved/deployed changes, snapshots | `/agent-audit`, `/audit`, `/design-agent`, `/implement` |
| `docs/MCP_REQUIREMENTS.md` | MCP tool gaps the analyst needs closed, as Linear-ready tickets | manual |

The planning documents are designed to be **synced to Day AI Pages** (`/sync-pages`) so the rest of the company sees the same source of truth.

## Agents are GTM automation

A core thesis the analyst and implementor operate on: **almost every active person should be running at least two Day AI agents.** One agent is a chat; two or more means real job functions have been delegated. Every seller's baseline is a **CRM Data Nerd** (keeps the customer record correct and complete) and a **Coach** (deep on every deal, fluent in the company's process, preps and follows up). An agent's **identity description is its definition** — the equivalent of an `.md` agent definition here — so a blank or generic description is an unconfigured agent. The full archetype playbook and quality rubric live in `.claude/agents/data-analyst.md`.

**Measuring value honestly:** the MCP confirms whether an agent is *well-built* (identity, skill craft, automation) and — via `get_skill_history` — whether each skill is *actually delivering*: firing recently, producing substantive (not hollow) output, and delivered (per the run's `notification.result`, never the channel config). The remaining unknown is engagement *depth* (does a human act on it?), which needs tools recorded in `docs/MCP_REQUIREMENTS.md`. Never assert "delivering value" from a schedule existing — read the run history.

---

## How the pieces fit

```
/setup        → connection + role check, identify people, scaffold the plan
/plan         → gtm-strategist + data-analyst build the planning layer
/agent-audit  → data-analyst scores how well you're using Day AI's agents + recommends
/design-agent → data-analyst + agent-implementor design one complete agent (identity + skills)
/audit        → data-analyst + agent-implementor diff the workspace vs. the plan
/implement    → agent-implementor invites people, tunes agents, deploys skills
/sync-pages   → push/pull planning docs to Day AI Pages
```

Two audit lenses: `/agent-audit` measures how well you're using Day AI's agents (independent of the plan); `/audit` measures how well the workspace delivers the plan. Read the plan, raise the bar on the agents, change the workspace, keep them in sync. That's the loop.
