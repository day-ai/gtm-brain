# The Instruction Architecture

Day AI composes instructions in four layers. Every rule your discovery surfaces belongs in exactly one of them, and most do-it-yourself setups fail here first: task instructions bloat the workspace layer, universal guardrails get buried in one skill and skip the other twelve. This doc is the sorting guide. `/discover` applies it at Gate 5; you should understand it before touching anything.

The layers **compose**; lower layers don't override higher ones, they add scope.

```
Workspace instruction     one record, every agent inherits it, everywhere
   └─ Agent               identity, personality, per-tool + per-event instructions, tier
        └─ Skill          the task prompt, on a trigger
             └─ Prompt    the chat message or slash command itself
```

## Layer 1: The workspace instruction

One editable record for the whole workspace. Every agent inherits it in everything it does, and it is most load-bearing in **chat**, where no skill prompt scopes the work.

- **What belongs here:** company terminology ("our stages are X → Y → Z"), universal guardrails ("never email customers directly"), the top 1-3 business priorities, norms for handing work to a human.
- **What doesn't:** anything task-shaped. Work that runs on a schedule or produces a deliverable is a skill.
- **The contract:** 3000-character cap, Owner/Admin-only writes, and `update` **replaces the entire text**. Always read the current text, merge, and write back the whole result. The cap is a feature: only rules that genuinely apply everywhere, always.
- Pre-signup, the draft lives at `rollouts/preflight/WORKSPACE_INSTRUCTION.md`, written to the cap.

## Layer 2: The agent

An agent's **identity description is its definition**: the equivalent of a system prompt. A blank or generic description is an unconfigured agent. Beyond identity: personality settings, per-tool instructions, per-event instructions (including the default chat instruction), schedules, and the **tier**, which gates which tools the agent can use and how many automated skills it can run.

- **Personal vs managed:** every agent belongs to a member. Personal agents are configured by their owner; managed agents are deployed and maintained by an Admin/Owner (from templates or by hand) for teammates. Fleet design decides which is which per person.
- **Delegation:** agents can hand work to other agents. Wire it explicitly in the fleet design; delegation changes what each agent's skills need to cover.
- **The one thing you can't automate:** creating an agent happens in the Day AI UI (it's a seat/billing step). Everything after creation (identity, skills) applies via MCP. Pre-signup fleet designs ship a creation card per agent for this step.

## Layer 3: The skill

A skill is a prompt with a trigger. Three trigger types:

- **SCHEDULE:** cron + timezone. Morning briefings, weekly syntheses.
- **EVENT:** fires on meeting ended, email received, deal stage changed, contact created, and similar.
- **On-demand:** no trigger; invoked by its slash command in chat.

Skills with a SCHEDULE or EVENT trigger are **automated skills** and consume the *target agent's* tier slots; budget them during fleet design, not at deploy time. Delivery defaults to email; **route to Slack deliberately**, because a briefing in an unread inbox trains people to ignore their agent. Scope is per-agent or workspace library; a team-wide capability is one MANAGED library skill, not N per-agent copies.

The craft bar for the prompt itself lives in `.claude/skills/write-skill/SKILL.md`. Skills reference shared Pages by title and id and read them at run time, never paste their content; that's what makes the living-guide flywheel work.

## Layer 4: The prompt

The chat message or slash command. It inherits everything above it. If you find yourself pasting the same context into chat repeatedly, that context belongs in a higher layer.

## The sorting test

For every rule discovery surfaces, ask: **who needs this, and when?**

| Answer | Layer |
|--------|-------|
| Everyone, always, everywhere | Workspace instruction (if it fits the cap's bar) |
| One agent, in everything it does | Agent identity or per-tool/per-event instruction |
| One recurring task | Skill prompt |
| Just this once | The chat prompt |

Similar lines appearing across several skills is normal, not a smell; the workspace instruction is not a home for shared task boilerplate.
