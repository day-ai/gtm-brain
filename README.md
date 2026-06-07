# GTM Brain

**A Claude Code harness for running your Day AI workspace like a GTM operating system.**

## Quickstart

1. **Clone** this repo and open the folder in [Claude Code](https://claude.com/claude-code).
2. **Authenticate** the Day AI MCP server (approve the `day-ai` server, complete the OAuth flow). You'll need to be an Owner or Admin of your workspace.
3. **Run `/setup`** — it checks the connection, learns who's in your workspace, and scaffolds your plan.

That's it. Details below.

---

This repo is a working example of how a go-to-market team can use [Claude Code](https://claude.com/claude-code) plus the **Day AI MCP server** to do two things well:

1. **Plan** — turn your company's goals, your revenue strategy, and the concrete outcomes you're driving toward into living documents that an agent understands.
2. **Implement** — translate that plan into real configuration in your Day AI workspace: the right people invited at the right roles, and every teammate's agent set up with thoughtful, role-specific skills that do real work on a schedule.

It is meant to be **cloned and adapted**. Nothing here is specific to one company — the planning documents are scaffolds for you to fill in, and the agents and skills know how to read your workspace and tailor everything to it.

---

## What this requires

Two things, both non-negotiable:

1. **The Day AI MCP server, authenticated.** Everything in this repo runs through it. Setup instructions are below.
2. **You must be an Owner or Admin of your Day AI workspace.** Most of what this harness does — reading and editing *other* teammates' agents, creating skills for them, inviting members, changing roles — requires the `USERS:manage` permission, which only Owners and Admins have. A Member can use the planning side, but the implementation side will return *"requires Admin or Owner"* errors. If you're not sure what role you are, run `/setup` — it checks first.

---

## Connect the Day AI MCP server

The MCP server lives at **`https://day.ai/api/mcp`** (streamable HTTP). It authenticates over OAuth — the first time a tool is called, you'll be prompted to authorize in the browser. That authorization resolves your workspace, your user, and the agent your token belongs to; you never pass any of those by hand.

This repo already ships a `.mcp.json` pointing at it:

```json
{
  "mcpServers": {
    "day-ai": { "type": "http", "url": "https://day.ai/api/mcp" }
  }
}
```

When you open this folder in Claude Code, you'll be asked to approve the `day-ai` MCP server. Approve it, then complete the OAuth flow. To verify the connection and your role in one step, run:

```
/setup
```

If you prefer to add it manually (or to your global config):

```
claude mcp add --transport http day-ai https://day.ai/api/mcp
```

---

## The model: a planning layer and an implementation layer

```
┌─────────────────────────── PLANNING LAYER ───────────────────────────┐
│  planning/COMPANY_PLAN.md   High-level goals, forecasts, plans         │
│  planning/STRATEGY.md       CRO-level revenue strategy & direction     │
│  planning/OUTCOMES.md       Concrete outcomes that serve the above     │
│  workspace/PEOPLE.md        Who's who in the workspace (the cast)      │
└───────────────────────────────────────────────────────────────────────┘
                                   │
                    the plan is the source of truth
                                   ▼
┌──────────────────────── IMPLEMENTATION LAYER ────────────────────────┐
│  Audit the current workspace → find the gap between plan and reality   │
│  Invite the right people at the right roles                            │
│  Configure each teammate's agent identity                              │
│  Deploy role-specific skills that do real work on a schedule           │
│  Sync planning docs to/from Day AI Pages                               │
└───────────────────────────────────────────────────────────────────────┘
```

You write (with an agent's help) what the business is trying to do. The harness then makes your Day AI workspace *reflect* that — and keeps it reflecting it as the plan evolves.

---

## The agents

Three subagents do the work. You rarely invoke them directly — the skills below orchestrate them — but they're defined in `.claude/agents/`:

| Agent | Role |
|-------|------|
| **`gtm-strategist`** | Builds and maintains the planning layer. Identifies the key players, interviews you to fill gaps, and runs through your workspace graph to ground the plan in what's actually there. |
| **`agent-implementor`** | The workhorse. Reads the plan, audits the workspace, and creates/updates agents and skills via the MCP. Writes every skill prompt to a high bar. |
| **`data-analyst`** | Grounds everything in reality — pulls the workspace's pipeline, contacts, meetings, and current agent/skill configuration so the plan and rollout are built on what the data actually supports. |

## The skills (slash commands)

| Command | What it does |
|---------|-------------|
| **`/setup`** | **Start here.** Verifies the MCP connection and your role, identifies the workspace and its people, and scaffolds your planning documents through a short interview. |
| **`/plan`** | Build or refresh the three planning-layer documents. Interview loop + workspace asset discovery. |
| **`/audit`** | Compare the current workspace (members, agents, skills) against the plan and produce a prioritized gap report. No changes are made. |
| **`/implement`** | Turn the plan and audit into real changes: invites, agent identity, and deployed skills. Always previews before it writes. |
| **`/write-skill`** | The teaching guide for authoring a single high-quality skill prompt. Read by the implementor before it writes anything. |
| **`/sync-pages`** | Sync the planning documents to/from Day AI Pages so the rest of your company can see them. |

---

## Quickstart

```
1.  Open this folder in Claude Code and approve the `day-ai` MCP server.
2.  Run  /setup            → connection check, who's-who, planning scaffolds
3.  Run  /plan             → fill in goals, strategy, and outcomes
4.  Run  /audit            → see the gap between the plan and the workspace
5.  Run  /implement        → invite people, tune agents, deploy skills
```

Re-run `/audit` and `/implement` whenever the plan changes. The plan is living; the workspace should track it.

---

## Repo map

```
gtm-brain/
├── README.md                 ← you are here
├── CLAUDE.md                 ← operating principles for every agent in this repo
├── .mcp.json                 ← Day AI MCP server config
├── .claude/
│   ├── agents/               ← gtm-strategist, agent-implementor, data-analyst
│   └── skills/               ← setup, plan, audit, implement, write-skill, sync-pages
├── planning/
│   ├── COMPANY_PLAN.md       ← layer 1: goals, forecasts, plans
│   ├── STRATEGY.md           ← layer 2: CRO-level strategy & direction
│   └── OUTCOMES.md           ← layer 3: concrete outcomes
├── workspace/
│   └── PEOPLE.md             ← who's who in the workspace
└── rollouts/                 ← per-rollout build artifacts the implementor writes
```

---

## A note on trust

This harness can change your live workspace — invite people, edit your teammates' agents, deploy skills that email and Slack them. That power is the point, but it demands care:

- **It previews before it writes.** Every `/implement` run shows you exactly what it will do and waits for your go-ahead.
- **It respects your teammates.** Skills it writes for other people are grounded in their actual role and data, never generic filler, and never expose internal strategy notes.
- **You are the Owner/Admin in the loop.** The agents propose; you approve. Read what they're about to deploy before you say yes.
