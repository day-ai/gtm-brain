# GTM Brain — Operating Guide

This repo is a GTM operating system. A human Owner/Admin and a set of agents work from shared planning documents and execute against a live **Day AI** workspace through the Day AI MCP server.

There are two layers, and they have a strict relationship:

- **The planning layer** (`planning/`, `workspace/PEOPLE.md`) is the source of truth for *what the business is trying to do and who is doing it*.
- **The implementation layer** makes the Day AI workspace *reflect* the plan — the right people, at the right roles, each with an agent configured to do their real work.

Plan first. Implement from the plan. When reality and the plan diverge, the job is to either change the workspace or update the plan — never to let the gap sit silently.

**Initiatives are the unit of work between the two.** An **initiative** (`initiatives/<slug>.md`) is a bounded, owned, time-boxed effort with a verifiable definition of success — it pulls from the plan and drives implementation work until its success criteria actually verify against the workspace. Initiatives sit *above* the fine-grained outcomes in `planning/OUTCOMES.md`: an outcome is atomic ("draft a follow-up after a call"); an initiative is the larger effort an outcome serves ("get the team running on Day AI by Q3"), realized through many outcomes, invites, agents, and skills. **`/start`** is the entrypoint that takes stock of every initiative and kicks off what each one needs. See [`initiatives/README.md`](initiatives/README.md) for the file format, status lifecycle (`NEW → IN_PROGRESS → SUCCEEDED`, plus `PAUSED`/`CANCELLED`), and the default `bootstrap-day-ai` initiative that ships with the repo.

---

## The three operator states

Every run starts by knowing which of three states the operator is in. `/start` detects it (via `manage_workspace_members` → `list_configuration`) and every other skill inherits the answer.

1. **Connected Owner/Admin.** The full harness. All workspace reads and writes go through the Day AI MCP; identity is implicit in the OAuth token — you never pass `workspaceId`, `userId`, or the current `assistantId`. You only pass `targetAssistantId` when acting on a *different* agent than your own. Cross-agent and member-management actions require the `USERS:manage` permission.
2. **Connected Member.** The planning side works; inviting people, editing teammates' agents, and creating skills for others are blocked. If a tool returns *"requires Admin or Owner,"* stop and tell the operator — don't design around it.
3. **Prospect (no workspace yet).** The pre-signup mode. The harness maps the GTM through `/discover` and the `map-your-gtm` initiative, and builds the deployable payload in `rollouts/preflight/`. Skills that need the workspace degrade the way `/sync-pages` does: state plainly what's unavailable and why, do the repo-side work, never pretend a write happened. Verification uses the pre-signup vocabulary — confirm from decisions and sign-offs, not from a doc existing.

In every state, the harness knows **who is operating it**: `/start` reads git config (and `gh auth status` when available) and records the operator in `workspace/PEOPLE.md` with a trust note. The harness acts with that person's hands.

---

## The Day AI MCP tools

This harness is built on the workspace-management tools plus the read-only graph tools the Day AI MCP exposes.

### Management tools (the core of this repo)

| Tool | Purpose | Key modes |
|------|---------|-----------|
| `mcp__day-ai__assistant_settings` | Inspect & edit agents — your own, or (Admin/Owner) any agent in the workspace | `read`, `update`, `list` |
| `mcp__day-ai__manage_skills` | Full skill lifecycle on an agent or in the workspace library, plus reading a skill's run history | `list`, `get`, `create`, `update`, `delete`, `reset_prompt`, `get_history` |
| `mcp__day-ai__manage_workspace_members` | Members, roles, invites, domain auto-invite, suggested invites | `list_configuration`, `invite_member`, `resend_invite`, `revoke_invite`, `update_invite_role`, `enable_auto_invite`, `disable_auto_invite`, `list_suggested_invites`, `navigate_to_billing` |
| `mcp__day-ai__manage_workspace_instructions` | The single workspace-wide instruction every agent inherits — the home for rules that apply across the board | `list_configuration`, `update` |

**Always start a member/invite task with `list_configuration`** — it returns the current members, roles, claimed domains, auto-invite config, and *what the caller is allowed to do*. Read `currentUser` before acting.

**Cross-agent / `list` modes are Admin/Owner-only.** Targeting an agent other than your own (`targetAssistantId`) requires it; so does `assistant_settings` `mode: "list"` and `manage_skills` reading another agent's skills.

**Tier limits are real.** Automated skills (those with a `SCHEDULE` or `EVENT` trigger) consume automated-skill slots governed by the *target agent's* tier. Over-budget creation is rejected with an explanation. When you create a skill for a teammate, the limit checks against *their* tier, not yours. Plan automations around the target agent's packaging; prefer one or two high-value scheduled skills over many.

**Result envelopes** are `{ result: {...} }` on success and `{ error: { message } }` on failure. Permission failures are explicit — surface them, don't retry blindly.

**Workspace instructions drive rules and consistency across the whole business.** The general workspace instruction is standing guidance every agent inherits everywhere it works. It matters most in **chat**, where there's no prompt scoping the work — it's what helps an agent route and act well: company context and terminology ("our stages are X → Y → Z"), universal guardrails ("never email customers directly"), norms for when to hand something to a human. Skill runs inherit it too, but it's less load-bearing there — a skill's prompt already defines its scope. It is **not** a home for task instructions: work that runs on a schedule or produces a deliverable is a skill (workspace-library if the whole team needs it), and similar lines appearing across several skills is normal, not a smell. The contract makes care mandatory: there is **one** editable record, writes are Owner/Admin-only, the text caps at **3000 characters**, and `update` **replaces the entire text**. Always `list_configuration` first, merge into the existing text, and write back the complete result — a naive update clobbers everything there. The cap is a feature: it keeps the record to the few rules that genuinely apply everywhere, always.

### Graph / read tools (for grounding)

Use the Day AI MCP's read-only graph tools to ground the plan and every skill prompt in what the workspace actually contains — pipeline, contacts, meetings, prior conversations. `search_objects` (general graph search) and `get_meeting_recording_context` (a specific meeting's full context) are the primary ones. Discover the rest from the connected tool list rather than assuming names. **Never fabricate a custom property, pipeline stage, or page that you haven't confirmed exists in the workspace.**

`manage_skills → get_history` (read a skill's recent runs — full transcript, firing times, and the per-run `notification` delivery block) is how the analyst confirms a skill is *actually delivering value*, not just configured. Use the run's `notification.delivered` boolean to confirm delivery — never the channel config. Admin/Owner can read any agent's skill history via `targetAssistantId`.

### Shared artifacts: pages, folders, and living guides

Some of what the harness deploys isn't agent config — it's **shared content agents work against**: playbooks, guides, dashboards. The MCP exposes `create_page` / `update_page` / `read_page` (rich HTML pages; `update_page` prefers targeted-edit and insert modes — full replace is a last resort that requires proof you read the current page) and `create_or_update_folder` (a folder created with `shareWithWorkspace: true` makes everything filed in it workspace-visible; deleting a folder unfiles its pages, never deletes them).

The signature pattern is the **living-guide flywheel**: a playbook (e.g. the discovery guide) lives as a workspace-shared Page, and skills on *different people's* agents form a loop around it — **consumer** skills on each seller's agent read the guide when prepping meetings, and a **producer** skill on the playbook owner's agent reviews recent calls against the guide and proposes targeted `update_page` edits. Because skills reference the page rather than embedding its content, every improvement propagates to everyone's prep the moment the page changes — no skill redeploys. The archetype and audit rubric live in `.claude/agents/data-analyst.md`; `/design-agent` proposes the pairing.

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
| `planning/COMPANY_PLAN.md` | Company goals, forecasts, plans (layer 1) | `/plan`, `/discover`, manual |
| `planning/STRATEGY.md` | CRO-level revenue strategy & direction (layer 2) | `/plan`, `/discover`, manual |
| `planning/OUTCOMES.md` | Concrete, fine-grained outcomes that serve layers 1–2 (layer 3) | `/plan`, `/discover`, manual |
| `initiatives/<slug>.md` | Bounded, owned, time-boxed efforts with verifiable success criteria and a status. The unit of work between the plan and implementation; realized through outcomes/agents/skills | `/start`, `/discover`, manual |
| `workspace/PEOPLE.md` | Who's who — roles, agents, focus, the operator, activation owners | `/start`, `/plan`, `/discover` |
| `workspace/TECH_STACK.md` | Every system a customer touches, owners, migration posture, data readiness | `/discover` |
| `workspace/PRIVACY.md` | Per-persona sharing tiers, exclusions, and the sign-off that gates all dataflow | `/discover` |
| `discovery/inbox/` | Source material the operator drops for discovery to read before it asks | operator |
| `rollouts/preflight/` | The deployable payload built pre-signup: instruction draft, properties, pages, invites, agent specs + creation cards, skills, import plan, enablement assets | `/discover` |
| `rollouts/<YYYY-MM-DD>-<slug>/` | Per-run artifacts: audit reports, agent specs, proposed/approved/deployed changes, snapshots | `/agent-audit`, `/audit`, `/design-agent`, `/implement` |
| `rollouts/health/` | The binding manifest and dated brain-health reports | `/brain-health` |
| `docs/INSTRUCTION_ARCHITECTURE.md` | Where every rule lives: workspace → agent → skill → prompt | shipped; read before configuring |
| `docs/MCP_REQUIREMENTS.md` | MCP tool gaps the analyst needs closed, as Linear-ready tickets | manual |

This data layer lives in the team's **own private GitHub repo** (made from the `day-ai/gtm-brain` template). That repo is the version-controlled source of truth the **operators** sync through — `git pull` before working, `git push` when the plan, an initiative, or `PEOPLE.md` changes. It's private because it holds strategy, forecasts, and candid notes about teammates. Three planes, kept distinct: the **private repo** (operators author and sync here), **Day AI Pages** (a published mirror of the plan and active initiatives the whole company reads, via `/sync-pages`), and the **Day AI workspace** (where it all executes, via the MCP). See the README's "Your team's GTM Brain repo."

## Agents are GTM automation

A core thesis the analyst and implementor operate on: **almost every active person should be running at least two Day AI agents.** One agent is a chat; two or more means real job functions have been delegated. Every seller's baseline is a **CRM Data Nerd** (keeps the customer record correct and complete) and a **Coach** (deep on every deal, fluent in the company's process, preps and follows up). An agent's **identity description is its definition** — the equivalent of an `.md` agent definition here — so a blank or generic description is an unconfigured agent. The full archetype playbook and quality rubric live in `.claude/agents/data-analyst.md`.

**Two is the default expectation, not a quota to hit.** Agents cost money — they consume seats, and their automated skills consume tier budget. So every recommendation to add an agent must carry its own **value-vs-cost case**: the job slice it delegates, the work product it would proactively produce, and the seat/tier it requires. An agent worth adding pays for itself many times over and that case is easy to make; an agent added to move a count is waste. Never recommend "add a second agent" without making the case, and surface the seat/tier cost (`navigate_to_billing` when a seat is needed) as part of the recommendation, not as a surprise at deploy time.

**Measuring value honestly:** the MCP confirms whether an agent is *well-built* (identity, skill craft, automation) and — via `manage_skills → get_history` — whether each skill is *actually delivering*: firing recently, producing substantive (not hollow) output, and delivered (per the run's `notification.delivered`, never the channel config). The remaining unknown is engagement *depth* (does a human act on it?), which needs tools recorded in `docs/MCP_REQUIREMENTS.md`. Never assert "delivering value" from a schedule existing — read the run history.

---

## Beyond skills: the Day AI SDK

Everything above runs *inside* Day AI. When an outcome needs something the workspace can't express — a custom internal app, a dashboard, a mobile view, a cron job running elsewhere, a mashup with another system — the **Day AI SDK** is the escape hatch: [github.com/day-ai/day-ai-sdk](https://github.com/day-ai/day-ai-sdk), public and MIT-licensed. It's a TypeScript client over the *same MCP tools this harness uses* (typed convenience methods plus raw `mcpCallTool()`, OAuth with auto-refresh, and a built-in pattern for handing the full toolkit to an LLM), shipped with example templates — desktop, mobile, Next.js, Vercel cron — that are meant to be cloned and reshaped.

Don't replicate its docs here. If someone wants to vibe-code a Day AI app, run **`/build-app`**: it knows what the SDK offers, clones it, and works from the nearest example template against the SDK's own `README.md`/`CLAUDE.md`/`SCHEMA.md`.

---

## How the pieces fit

```
/start        → detect operator state, take stock of initiatives, kick off what each one needs
/discover     → guided discovery: map the GTM (works with no workspace), scope the first win,
                build the preflight payload
/plan         → gtm-strategist + data-analyst build the planning layer
/agent-audit  → data-analyst scores how well you're using Day AI's agents + recommends
/design-agent → data-analyst + agent-implementor design one complete agent (identity + skills)
/audit        → data-analyst + agent-implementor diff the workspace vs. the plan
/implement    → agent-implementor invites people, tunes agents, deploys skills;
                preflight mode applies rollouts/preflight/ at connect time
/brain-health → binding manifest + drift report; proposals only, never applies
/sync-pages   → push/pull planning docs to Day AI Pages
/build-app    → vibe-code a custom app on the public Day AI SDK
```

`/start` is the standing entrypoint: it reads `initiatives/`, reports each initiative's progress against its verifiable success criteria, and hands off to the skills above to do the work. On a fresh clone with no workspace, the lead initiative is `map-your-gtm`; on a fresh connected clone it's `bootstrap-day-ai`. Two audit lenses: `/agent-audit` measures how well you're using Day AI's agents (independent of the plan); `/audit` measures how well the workspace delivers the plan. Read the plan, raise the bar on the agents, change the workspace, keep them in sync. That's the loop.

**Three standing loops keep the brain alive:** the living-guide flywheel keeps shared content improving (producers propose page edits from call evidence), a scheduled `/audit` keeps the workspace matching the plan, and `/brain-health` keeps the brain's own bindings from rotting as the org changes. All three propose; the operator applies.

**The posture the whole harness earns:** treat the brain the way a good founder treats a paid consultant — give it everything, answer its questions honestly, and listen to what it says. The harness holds up its end by doing the homework before asking (discovery reads before it interviews), citing evidence for every claim, and saying plainly what it can't verify.
