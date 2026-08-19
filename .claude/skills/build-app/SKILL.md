---
name: build-app
description: >-
  Vibe-code a custom app or integration on top of Day AI using the public Day
  AI SDK (github.com/day-ai/day-ai-sdk) — internal tools, dashboards, mashups,
  automated workflows, anything a skill or Page can't express. Knows what the
  SDK offers, how to get started, and which example template to clone. Usage:
  /build-app [what you want to build]
metadata:
  internal: true
---

# /build-app

Sometimes an outcome can't be delivered by an agent skill or a Page — it needs a real interface, an external automation, or a mashup with another system. A deal-room dashboard. A meeting-prep mobile view. A cron job that pushes pipeline data somewhere else. That's what the **Day AI SDK** is for, and this skill is the bridge to it.

$ARGUMENTS

---

## What the SDK is

**[github.com/day-ai/day-ai-sdk](https://github.com/day-ai/day-ai-sdk)** — public, MIT-licensed, TypeScript. The pitch: *anything you can do in Day AI, you can do with the SDK.* It speaks to the same MCP tools this harness uses (`search_objects`, `create_or_update_opportunity`, `get_meeting_recording_context`, `manage_skills`, notifications, imports, prospecting, …), so everything you know about the workspace graph transfers directly.

It offers a lot:

- **A typed client** (`DayAIClient`) with convenience methods — `search()`, `createPerson()`, `createOpportunity()`, `sendNotification()` — plus `mcpCallTool()` as the escape hatch to *any* MCP tool.
- **OAuth 2.0 with auto token refresh** — `yarn oauth:setup` registers a client, runs the browser flow, and saves credentials. No manual token wrangling.
- **The LLM pattern built in** — `mcpListTools()` hands the full Day AI toolkit to Claude (or any LLM) as tool definitions, so an AI-powered app gets the whole customer memory as its toolkit in a few lines.
- **Example templates that are meant to be cloned and reshaped**, not just read:

| Template | What it is | Stack |
|----------|-----------|-------|
| `examples/desktop/` | Notes app with AI chat + CRM context | Electron, React, Claude SDK |
| `examples/desktop-claude-agent-sdk/` | Desktop app on the Claude Agent SDK | Electron, React, Agent SDK |
| `examples/community-builder/` | Community management tool | Next.js |
| `examples/mobile/` | Mobile app with OAuth deep linking | React Native, Expo |
| `examples/vercel-weather-cron/` | Automated daily workflow → Day AI notification | Next.js, Vercel Cron |

One caveat to carry: **which MCP tools the SDK can call depends on the authorizing user's assistant tier** (free → turbo → professional → executive, cumulative). Check the README's tier table before designing around a tool.

## How to work with it

Don't replicate the SDK's documentation here — clone it and work inside it. The cloned repo carries its own `README.md`, `CLAUDE.md` (context written for Claude sessions), and `SCHEMA.md` (full tool and object-schema reference). Those are the source of truth; read them there.

1. **Start from what they want, not from the stack.** Ask (or take from `$ARGUMENTS`) what the app should *do* and who will use it. Ground it the way every skill is grounded: real pipeline, real meetings, real people from `workspace/PEOPLE.md`. If it traces to an initiative or outcome, say which.
2. **Get the SDK.** If a local checkout already exists (e.g. a sibling `day-ai-sdk/` directory), offer to work from that; otherwise clone:
   ```bash
   git clone https://github.com/day-ai/day-ai-sdk
   cd day-ai-sdk
   yarn install && yarn build
   cp .env.example .env   # set INTEGRATION_NAME
   yarn oauth:setup
   ```
3. **Pick the nearest example template** from the table above and build from it — cloning a working template beats scaffolding from scratch. `cd` into it and develop there; its README has the run command.
4. **Read `SCHEMA.md` before inventing a query.** The same grounding rule as everywhere in this repo: never fabricate a property, object type, or tool that you haven't confirmed exists.
5. **Prototype the data first.** Before building UI, verify the queries return what you expect — the `examples/*.ts` one-off scripts (`mcp-tool-example.ts`, `recent-meetings-context.ts`, `pagination-example.ts`) show the pattern.

## When to reach for it (and when not to)

- **Reach for the SDK** when the outcome needs a UI, runs outside Day AI (external cron, another system's webhook), or mashes Day AI data up with something the workspace can't see.
- **Don't reach for it** when an agent skill, a Page, or a view already delivers the outcome — those are cheaper to build, live where the team already works, and need no hosting or credentials. The SDK is the escape hatch, not the default.

If you build something the team will rely on, record it: where it lives, what it does, and which initiative/outcome it serves — a line in the relevant initiative file or `planning/OUTCOMES.md` is enough.
