<p align="center">
  <img src="assets/gtm-brain-ribbons.png" alt="GTM Brain" width="240">
</p>

<h1 align="center">GTM Brain</h1>

<p align="center"><strong>A Claude Code harness for running your Day AI workspace like a GTM operating system.</strong></p>

## Quickstart

1. **Make it your team's private GTM Brain repo.** Do this once, as one of the
   operators who will run it. Install the setup skill:

   ```sh
   npx skills add day-ai/gtm-brain --global
   ```

   Then run the `/setup-gtm-brain` skill. It creates the private team repo with
   the complete harness and retains `day-ai/gtm-brain` as `upstream`. Add the
   people who will run the harness—typically your CRO, RevOps lead, and chief
   of staff—as GitHub collaborators. Keep the repo private: it holds strategy,
   forecasts, and candid operating notes.

2. **Open your team's repo** in your coding agent. If another operator already
   created it, clone that private repository instead; do not run the setup
   skill again.
3. **Authenticate** the Day AI MCP server. Approve the `day-ai` server from
   `.mcp.json` and complete the OAuth flow. You'll need Owner or Admin access
   for the full implementation workflow.
4. **Run the `/start` skill.** It checks the connection, takes stock of the
   initiatives, and on a fresh repository begins the default bootstrap work:
   learn who's in your workspace and scaffold your plan.

That's it. Details below.

---

This repo is a working example of how a go-to-market team can use [Claude Code](https://claude.com/claude-code) plus the **Day AI MCP server** to do three things well:

1. **Plan** — turn your company's goals, your revenue strategy, and the concrete outcomes you're driving toward into living documents that an agent understands.
2. **Audit** — measure how much value you're actually getting from your Day AI agents and the workspace today, and surface the gap: who's missing, who needs the agents they don't have, and which skills and identities are weak.
3. **Implement** — translate that plan into real configuration in your Day AI workspace: the right people invited at the right roles, and every teammate's agent set up with thoughtful, role-specific skills that do real work on a schedule.

It is meant to be **cloned and adapted**. Nothing here is specific to one company — the planning documents are scaffolds for you to fill in, and the agents and skills know how to read your workspace and tailor everything to it.

---

## What this requires

Two things, both non-negotiable:

1. **The Day AI MCP server, authenticated.** Everything in this repo runs through it. Setup instructions are below.
2. **The full implementation workflow requires Owner or Admin.** Reading and editing *other* teammates' agents, creating skills for them, inviting members, and changing roles require `USERS:manage`. A Member can still use the planning side and configure their own agent. If you're not sure what role you have, run the `/start` skill—it checks first.

---

## Connect the Day AI MCP server

The MCP server lives at **`https://day.ai/api/mcp`** (streamable HTTP). It authenticates over OAuth — the first time a tool is called, you'll be prompted to authorize in the browser. That authorization resolves your workspace, your user, and the agent your token belongs to; you never pass any of those by hand.

This repository already ships a `.mcp.json` pointing at the Day AI MCP server:

```json
{
  "mcpServers": {
    "day-ai": { "type": "http", "url": "https://day.ai/api/mcp" }
  }
}
```

When you open this folder in Claude Code, approve the `day-ai` MCP server and
complete the OAuth flow. To verify the connection and your role, run `/start`.

If you prefer to add the server manually (or to your global configuration):

```sh
claude mcp add --transport http day-ai https://day.ai/api/mcp
```

---

## Your team's GTM Brain repo

GTM Brain runs across **three planes**, and it's worth keeping them straight:

| Plane | What it holds | Who it's for | How it syncs |
|-------|---------------|--------------|--------------|
| **Your private GitHub repo** | The authored source of truth: the plan, the initiatives, `PEOPLE.md` | The **operators** who run the harness | `git push` / `git pull` |
| **Day AI Pages** | A published, readable mirror of the plan and active initiatives | The **whole company** | `/sync-pages` |
| **Your Day AI workspace** | The live execution surface: members, agents, skills | Everyone, via their agents | the Day AI MCP (`/implement`) |

The public `day-ai/gtm-brain` repo is the harness source, not an enabled GitHub template. The guided setup skill creates the team repo with the correct history and remotes. The result should follow these rules:

1. **Keep the team repo private.** The planning layer contains revenue strategy, forecasts, and candid notes about teammates that don't belong in a public repo.
2. **Add your operators as collaborators.** The people who actually run the harness — typically a small group: CRO, RevOps, chief of staff. They each clone the repo and work against the same `main`.
3. **Keep it in sync like any shared repo.** `git pull` before you start, `git push` when you've updated the plan, an initiative, or `PEOPLE.md`. The repo is how operators stay aligned on *what the business is doing*; Day AI Pages is how the rest of the company reads it; the workspace is where it executes.
4. **Pull harness improvements (optional).** Keep `https://github.com/day-ai/gtm-brain.git` as `upstream`, so new skills and agent definitions can be merged with `git pull upstream main`.

> The default `bootstrap-day-ai` initiative tracks this: "the team's private repo exists and the operators can sync" is one of its success criteria, so the `/start` skill checks that it is actually set up.

---

## The model: plan → initiatives → implementation

```
┌─────────────────────────── PLANNING LAYER ───────────────────────────┐
│  planning/COMPANY_PLAN.md   High-level goals, forecasts, plans         │
│  planning/STRATEGY.md       CRO-level revenue strategy & direction     │
│  planning/OUTCOMES.md       Concrete, fine-grained outcomes            │
│  workspace/PEOPLE.md        Who's who in the workspace (the cast)      │
└───────────────────────────────────────────────────────────────────────┘
                                   │
                    standing intent feeds bounded efforts
                                   ▼
┌────────────────────────────── INITIATIVES ───────────────────────────┐
│  initiatives/<slug>.md      Bounded, owned, time-boxed efforts with    │
│                             verifiable success criteria + a status     │
│                             (NEW · IN_PROGRESS · PAUSED · CANCELLED ·   │
│                             SUCCEEDED). The unit of work.              │
└───────────────────────────────────────────────────────────────────────┘
                                   │
                   /start takes stock and kicks off the work
                                   ▼
┌──────────────────────── IMPLEMENTATION LAYER ────────────────────────┐
│  Audit how well you're using Day AI's agents today (the agent-value    │
│    review) and the gap between the plan and reality                    │
│  Invite the right people at the right roles                            │
│  Give each person the agents they should have, with strong identities  │
│  Deploy role-specific skills that do real work on a schedule           │
│  Sync planning docs to/from Day AI Pages                               │
└───────────────────────────────────────────────────────────────────────┘
```

You write (with an agent's help) what the business is trying to do. You break that into **initiatives** — bounded efforts with a clear, checkable definition of success and someone accountable. The harness then makes your Day AI workspace *reflect* the plan and drive each initiative to done — and keeps it reflecting as the plan evolves.

**Initiatives vs. outcomes.** An outcome (`planning/OUTCOMES.md`) is atomic — "draft a follow-up after a call." An initiative is the larger, owned effort an outcome serves — "get the team running on Day AI by end of Q3" — realized through many outcomes, invites, agents, and skills. Every workspace starts with one default initiative, `bootstrap-day-ai`: get connected, get the people in, get each person on the agents they should have, get the plan real and synced. `/start` runs it first. See [`initiatives/README.md`](initiatives/README.md) for the file format and status lifecycle.

---

## Day AI agents are GTM automation

The point of Day AI isn't a smarter chatbot you talk to all day. It's **automation**: you carve slices of your job into agent-shaped job descriptions and hand them over, and the agents produce work product proactively — prep, drafts, clean records, coaching — without anyone having to ask.

That has a sharp implication this harness is built around: **almost every active person should be running at least two agents.** One agent is a chat. Two or more means you've actually started delegating job functions. For sellers specifically, the baseline is two:

- **A CRM Data Nerd** — keeps every opportunity, note, and context object correct and complete from what actually happened in conversations. Works whether you run on Day AI, HubSpot, or Salesforce. This is the agent that ends "the CRM is always out of date."
- **A Coach** — goes deep on every deal: patterns, blockers, what's working on other reps' deals, fluent in *your* pipeline and process. Preps you before meetings, drafts emails to unstick deals, follows up after calls.

`/agent-audit` measures how far a workspace is from that bar and hands you the path to close it. `/design-agent` builds the agents. The whole harness exists to make the value of Day AI's agents real and visible — and most teams are capturing a fraction of it.

> **A note on measuring value.** The analyst confirms an agent is *well-built* (identity, skill craft, automation) **and** *actually delivering* — it reads each skill's real run history via `manage_skills → get_history` to check it's firing, producing substantive (not hollow) output, and being delivered. The one thing it still can't see is engagement *depth* — whether a human acts on the output — which needs a couple of MCP tools that don't exist yet. Those remaining gaps are written up as ready-to-file Linear tickets in [`docs/MCP_REQUIREMENTS.md`](docs/MCP_REQUIREMENTS.md), and the analyst is explicit about the line between what it can and can't confirm.

---

## The agents

Three subagents do the work. You rarely invoke them directly—the skills below orchestrate them. They are defined in `.claude/agents/`:

| Agent | Role |
|-------|------|
| **`gtm-strategist`** | Builds and maintains the planning layer. Identifies the key players, interviews you to fill gaps, and runs through your workspace graph to ground the plan in what's actually there. |
| **`agent-implementor`** | The workhorse. Reads the plan, audits the workspace, and creates/updates agents and skills via the MCP. Writes every skill prompt to a high bar. |
| **`data-analyst`** | The agent-value analyst. Grounds the plan in reality *and* evaluates how much value you're actually getting from your Day AI agents — who should be in the workspace, who's missing the agents they need, and whether the skills and identities are any good. Recommends; never executes. |

## The skills

| Skill | What it does |
|-------|--------------|
| **/start** | **Start here, every time.** Verifies the MCP connection and your role, takes stock of every initiative, and kicks off what the active ones need. |
| **/plan** | Build or refresh the three planning-layer documents. |
| **/agent-audit** | Score how well the workspace uses Day AI agents and produce a prioritized recommendation set. No changes are made. |
| **/design-agent** | Design one complete, deployment-ready agent for a person. |
| **/audit** | Compare the current workspace against the plan. No changes are made. |
| **/implement** | Turn the approved plan and designs into real changes. Always previews before it writes. |
| **/write-skill** | The teaching guide for authoring a high-quality skill prompt. |
| **/sync-pages** | Sync planning documents to/from Day AI Pages. |
| **/build-app** | Build a custom app or integration on the public [Day AI SDK](https://github.com/day-ai/day-ai-sdk). |

---

## The full flow

1. Open this folder in Claude Code and authenticate the `day-ai` MCP server.
2. Run **/start** for the connection check, initiative board, and next move. On a fresh clone it performs the light first pass for `bootstrap-day-ai`.
3. Run **/plan** to fill in goals, strategy, and outcomes.
4. Run **/agent-audit** and **/audit** for the agent-value and plan-alignment views.
5. Run **/design-agent** for missing agents.
6. Run **/implement** to preview and, after approval, deploy invites, agent changes, and skills.

Come back to **/start** whenever you sit down to work. It is the standing entrypoint that tells you where every initiative stands and what to do next, not just a first-run command. **/agent-audit** and **/audit** are two lenses: one on how well you are using Day AI and one on how well the workspace delivers your plan. Re-run them—and **/implement**—whenever the plan or team changes.

---

## Repo map

```
gtm-brain/
├── README.md                 ← you are here
├── CLAUDE.md                 ← Claude Code operating principles
├── .mcp.json                 ← Claude Code Day AI MCP config
├── .claude/
│   ├── agents/               ← gtm-strategist, agent-implementor, data-analyst
│   └── skills/               ← /start, /plan, /agent-audit, /design-agent, /audit,
│                                /implement, /write-skill, /sync-pages, /build-app
├── initiatives/
│   ├── README.md             ← what an initiative is: schema, statuses, lifecycle
│   ├── TEMPLATE.md           ← copy this to start a new initiative
│   └── bootstrap-day-ai.md   ← the default first-run initiative
├── planning/
│   ├── COMPANY_PLAN.md       ← layer 1: goals, forecasts, plans
│   ├── STRATEGY.md           ← layer 2: CRO-level strategy & direction
│   └── OUTCOMES.md           ← layer 3: concrete, fine-grained outcomes
├── workspace/
│   └── PEOPLE.md             ← who's who in the workspace
├── docs/
│   └── MCP_REQUIREMENTS.md   ← MCP tool gaps the analyst needs (Linear-ready)
└── rollouts/                 ← audit reports, agent specs, and deploy snapshots
```

## A note on trust

This harness can change your live workspace — invite people, edit your teammates' agents, deploy skills that email and Slack them. That power is the point, but it demands care:

- **It previews before it writes.** Every `/implement` run shows you exactly what it will do and waits for your go-ahead.
- **It respects your teammates.** Skills it writes for other people are grounded in their actual role and data, never generic filler, and never expose internal strategy notes.
- **You are the Owner/Admin in the loop.** The agents propose; you approve. Read what they're about to deploy before you say yes.
