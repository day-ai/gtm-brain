---
name: setup
description: First-run entry point for GTM Brain. Verifies the Day AI MCP connection and the operator's role, identifies the workspace and its people, and scaffolds the planning documents through a short interview. Run this once when you clone the repo. Usage: /setup
---

# /setup

The front door. Run this once after cloning the repo and connecting the Day AI MCP server. It confirms everything works, learns who's in the workspace, and gets the planning layer started — without trying to do everything at once.

$ARGUMENTS

By the end of `/setup`, the operator should have: a confirmed connection, a clear statement of their role and what that allows, a populated `workspace/PEOPLE.md`, and three planning documents that are non-empty and coherent (even if rough). Depth comes later via `/plan`.

---

## Step 1 — Verify the connection and the role

This is a hard gate. Do it first.

Call `manage_workspace_members` with `action: "list_configuration"`.

- **If the call fails** (no MCP, not authorized): stop. Tell the operator the `day-ai` MCP server isn't connected or authorized, and point them at the README's "Connect the Day AI MCP server" section. Don't proceed.
- **If it succeeds**, read `currentUser.roleName` and report it plainly:
  - **Owner or Admin** → "You're an {Owner/Admin}. You can do everything in this harness: read and edit any teammate's agent, create skills for them, and manage members." Proceed.
  - **Member** → "You're a Member. You can build the planning layer and configure *your own* agent, but inviting people, editing teammates' agents, and creating skills for others all require Admin or Owner — those will be blocked. To use the full harness, ask an Owner to promote you." Proceed, but set expectations that `/implement` will be limited.

Report the basics from `list_configuration`: workspace name, member count by role, claimed domains.

---

## Step 2 — Identify the cast

Spawn the **`gtm-strategist`** subagent to build `workspace/PEOPLE.md`:

```
Build workspace/PEOPLE.md for this workspace. Use manage_workspace_members → list_configuration
for the roster and roles, assistant_settings → mode: "list" to map each person to their agent,
and list_suggested_invites for people in the CRM who aren't members yet. For key players whose
role isn't obvious, do light graph reconnaissance. Fill the template already in workspace/PEOPLE.md.
Mark anything you inferred so the operator can confirm. Return the cast summary and a short list of
people you couldn't confidently place.
```

When it returns, show the operator the cast and ask them to confirm or correct the placements — especially roles and who owns what. This is the first turn of the interview loop; keep it light.

---

## Step 3 — Scaffold the plan

Spawn the **`gtm-strategist`** again (or continue it) to do a light first pass at the three planning documents:

```
Do a first, light pass at the planning layer. Do quick graph reconnaissance (pipeline shape,
recent meetings, coverage) to inform your questions, then run a short interview — one question at
a time, each with your best guess — to capture just enough to make planning/COMPANY_PLAN.md,
planning/STRATEGY.md, and planning/OUTCOMES.md non-empty and coherent. Lock Layer 1 (goal) before
Layer 2 (strategy) before Layer 3 (outcomes). Leave anything genuinely unknown as a marked TODO
rather than padding. Keep each Layer-3 outcome tied to an owner from PEOPLE.md and, where it implies
a workspace change, note what it becomes (a skill, an agent edit, an invite).
```

Run the interview in the main thread so the operator can answer in real time. The goal is a coherent skeleton, not a finished plan.

---

## Step 4 — Orient the operator

Close by showing the operator where they are and what's next:

```markdown
## GTM Brain is set up

**Workspace:** {name}  ·  **You:** {role} ({what that allows})
**Cast:** {N} people in workspace/PEOPLE.md ({N} with agents, {N} suggested invites)
**Plan:** scaffolded — {N} open questions marked TODO

### What's next
- `/plan`      → deepen the planning layer; resolve the {N} open TODOs
- `/audit`     → see the gap between the plan and the workspace today
- `/implement` → invite people, tune agents, deploy skills (Owner/Admin)
- `/sync-pages`→ publish the plan to Day AI Pages for the rest of the company

The plan in `planning/` is yours to edit directly anytime — the agents read it as the source of truth.
```

---

## Notes

- **Don't deploy anything in `/setup`.** No invites, no agent edits, no skills. Setup builds understanding and the plan only. `/implement` is where changes happen.
- **Keep it short.** Setup should feel like a 10-minute orientation, not an exhaustive planning session. Resist the urge to fill every TODO now — that's what `/plan` is for.
- If the operator is a Member, still complete all four steps; just be honest in Step 1 and Step 4 about what `/implement` won't be able to do for them.
