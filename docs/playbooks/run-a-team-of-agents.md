# Playbook: run a team of agents (the Chief of Staff pattern)

> Used by `/design-agent` (the Chief of Staff archetype and the specialist
> contract), `/agent-audit` (anyone who owns two or more agents), and
> `/discover` Gate 5 (fleet design). The platform facts are in
> [`../PRODUCT_FACTS.md`](../PRODUCT_FACTS.md#delegation-a-team-of-agents).
> The skeleton prompt is in
> [`../../.claude/skills/design-agent/chief-of-staff.md`](../../.claude/skills/design-agent/chief-of-staff.md).

Each agent you add should own one job. Without a coordinator, each one also
messages you on its own, and every result waits for you to read it. Day AI's
guide calls this "bot sitting". Fix it with a **Chief of Staff (CoS)**:
one agent you talk to. It hands each piece of your work to the agent that
owns it, gathers what comes back, and brings you only the decisions. For the
why, link [How to organize a team of GTM AI
agents](https://day.ai/guides/organize-a-team-of-gtm-ai-agents). This
playbook covers the how and the limits.

## Step 1: Count the agents one person owns

A CoS coordinates **one user's own agents**. Delegation works only between
agents with the same owner, whatever the person's role. So count per person,
never per team.

| Agents the person owns | Shape | Cost |
|------------------------|-------|------|
| 1 | **No CoS.** That agent is already the single interface. If it's struggling, it's doing two jobs: split it before adding coordination. | none |
| 2–3 | **Light CoS.** Promote the agent the person talks to most. It gets a routing section in its description and one daily digest skill. There are no Team Brief pages and no publish schedule. | usually free: one slot on an existing agent |
| 4 | **Your call.** Go light until the other agents' scheduled work no longer fits one digest, or decisions start getting lost. Then go full. | |
| 5+ | **Full CoS.** A dedicated coordinating agent. Each specialist publishes to a Team Brief page, and the CoS runs a briefing, a decision queue, and an agent-health check. | one more agent, plus slots on every specialist (Step 4) |

A sales leader whose reps own the agents doesn't get a CoS over those agents,
because their agent can't delegate to them. That leader's agents can still
**read** pages the reps' agents publish to a workspace-shared folder, so a
team rollup is a page-reading skill, not delegation.

## Step 2: Fix the platform limits into the design

These limits shape every choice below. Each one is in `PRODUCT_FACTS.md`.

- **Only the top-level run fans out.** A delegated specialist can't delegate
  again. Specialists never hand work to each other mid-task. When a
  specialist needs another one, it writes that into its result (`NEEDS
  <agent>: <what>`), and the CoS routes it.
- **No delegation from a thread that started in Slack.** The CoS briefing can
  be *delivered* to Slack, but "go work on this" requests belong in the
  CoS's chat in Day AI. Write this into the CoS description, so a request
  that arrives in a Slack thread gets an honest answer: the CoS answers what
  it can itself and says the fan-out needs Day AI chat.
- **The receiving agent sees only what you pass it:** the prompt, up to 8
  attached records, the user's standing memory, and its own skills. The
  delegation prompt has to stand on its own.
- **Results come back later.** A delegation returns immediately, and its
  result posts into the asking chat when the work is done (up to about an
  hour later). A scheduled briefing *can* delegate and still deliver after the
  results land, but a briefing that waits on five fresh analyses every
  morning is slow and fragile. Have specialists publish ahead of time (Step
  5) and delegate only for the gaps.
- **Slots are per agent.** The CoS's skills use the CoS's slots, and each
  specialist's scheduled publish uses that specialist's slots.

## Step 3: Build the routing table from the person's real agents

The routing table is the core of the CoS description: which agent owns which
domain, and which of its skills to call. Generate it; never hand-write it
from memory.

1. `assistant_settings → list` gives the agents the person owns.
2. For each agent, `manage_skills → list` with `targetAssistantId` gives its
   skills: name, `slashCommand`, trigger, enabled.
3. Write one row per agent:

   | Domain (what it owns) | Agent | Route here when… | Skills to target |
   |-----------------------|-------|------------------|------------------|
   | {job slice, from the agent's description} | {name, title} | {the asks that belong here} | `/{slash}` for {purpose}; … |

   Also add one row for what stays with the CoS itself: scheduling, the
   briefing, and quick lookups that need a single check.
4. **Name agents and slash commands, not ids.** The agent roster already
   shows the CoS every owned agent's `skillId`, and the CoS resolves it at
   run time. An id pasted into a description breaks the day a skill is
   recreated. (Records are different: pass those by full id, see the
   delegation brief below.)
5. **Check the lanes.** If two agents claim the same domain, fix the overlap
   in their descriptions before writing the table. A CoS can't route around
   two agents that both think they own forecasting.

When an agent or skill is added, renamed, or removed, regenerate the table.
`/brain-health` treats the table as a binding.

## Step 4: Budget the slots

Count every agent's automated skills, not just the CoS's.

- **Light CoS:** one slot on the promoted agent for the daily digest. Every
  other agent keeps the scheduled skills it already runs.
- **Full CoS:** on the CoS, the morning briefing, and optionally an
  end-of-day decision digest: 1–2 slots. A Turbo CoS (2 slots) is enough for
  that. Use Professional if the briefing carries charts (`create_chart` and
  the widget tools are Professional and up) or if the CoS runs more scheduled
  work. On each specialist, one scheduled publish per cadence it owns, such
  as an evening sweep or a weekly review, each on that specialist's own
  slots.
- Show the math per agent: slots used of max (`assistant_settings → list`
  returns `automatedSkillSlots`). If a specialist is full, its new publish
  replaces a scheduled skill it no longer needs. It doesn't trigger an
  upgrade by default.
- A dedicated CoS is one more agent on the bill. Price it per CLAUDE.md's
  pricing rules, and make the value case in the spec: five agents messaging
  one person separately is the cost it removes.

## Step 5: The specialist contract

Under a CoS, every other agent follows the same contract. Write it into each
specialist's description (`/design-agent` does this). These are the
conventions the CoS depends on:

- **Opening:** "You are {Name}, {Person}'s {Role}." Then GOALS (outcomes,
  not tasks), RESPONSIBILITIES (what it owns and what it publishes), and
  BOUNDARIES (what it never does, such as edit records it only recommends on,
  or write customer-facing email outside its lane).
- **One lane, no overlap** with the person's other agents. The lane matches
  the routing table row.
- **It doesn't message the person on a schedule.** Only the CoS sends
  scheduled messages to the person. A specialist's scheduled skills publish
  instead:
  - **Full:** a dated section on its own Team Brief page, by a fixed time
    before the CoS briefing.
  - **Light:** the run itself is the output. The skill's final message opens
    with a 2–4 line summary, because the CoS reads it through `get_history`,
    whose previews are truncated.

  Urgent, time-boxed alerts (a deal-desk deadline today) may still go to
  the person directly, when the specialist's description says which ones.
- **Tag decisions `[APPROVE]`** with the evidence and the exact change
  proposed, so the CoS can lift them into the decision queue unchanged.
- **Flag hand-offs, don't make them:** `NEEDS <agent>: <what>`. In its own
  top-level runs a specialist *can* delegate, but leaving routing to the CoS
  keeps one record of who asked whom.
- **Label ownership** before discussing someone else's account or
  opportunity (the ownership rule, Step 6).

## Step 6: What the CoS does

The skeleton prompt in `chief-of-staff.md` covers all of this. A light CoS
uses rules 1–4. A full CoS uses all eight.

1. **Single interface.** Only the CoS sends the person scheduled messages.
   It never does a specialist's analysis itself.
2. **Delegate first.** Break the ask into parts and route each part by the
   table, targeting the specialist's skill when one fits. Send the parts in
   parallel in the same turn. Each delegation brief stands on its own:
   - every record attached by id (`objects`, up to 8) or written as a full
     id, never a short link or a truncated id;
   - why the work matters and what the result is for;
   - the constraints;
   - the record's owner, when it isn't the person.

   Then merge the results into **one answer**:
   - **flag disagreements** between specialists; never quietly reconcile
     them;
   - end with a **routing receipt**: "asked {agent} for A, {agent} for B";
   - while results are still out, say what's coming and from whom.

   Bring genuine decisions back mid-task, and nothing else. Never do the
   analysis and then ask "should I check with {agent}?" Keep trivial
   lookups (a single check) inline.
3. **Daily digest or briefing.**
   - **Light:** a digest. For each other agent, read its most recent
     scheduled runs (`manage_skills → get_history`, `limit` 1–2), then lead
     with decisions, then one line per agent, then **MISSING** (a run that
     should have fired and didn't). Use `get_thread_context` on a run's
     thread only when the preview isn't enough.
   - **Full:** a briefing that reads every Team Brief section first, then
     **DEADLINES** (external deadlines pinned at the top), **DECISIONS**
     (`[APPROVE]` items with evidence), **HEADLINES** (one line per
     specialist), **MISSING** (an expected section that didn't publish is a
     headline in its own right; the CoS keeps a per-weekday list of which
     sections to expect), and **AGENT HEALTH**.
4. **Ownership label and informed bypass.** Before any content about an
   account or opportunity, label it `[YOURS]`, or `[NOT YOURS — owned by
   {owner}]`. Reading is free once labeled. Any action on someone else's
   record stops for an explicit confirmation. When the person asks to cross
   a soft rule, never refuse and never silently comply:
   - name the rule;
   - ask "are you sure?";
   - proceed only on their stated reason, echoed back and logged.

   Hard rules (pricing bands, legal terms, whatever the company names) have
   no override; they escalate to the named owner. Pass the owner down in
   every delegation.
5. **Decision queue (full).** An `[APPROVE]` item surfaces once with
   evidence and is re-raised in every rollup until it's resolved. Asks that
   need a person outside the team stay queued until they're scheduled or
   declined.
6. **Monitor, don't self-heal (full).** Agent health means:
   - an expected section missing;
   - a specialist reporting BLOCKED;
   - output drifting from its description;
   - an agent silent for 2+ business days;
   - a skill disabled, or a scheduled skill that didn't fire (`get_history`).

   The CoS proposes each fix as an `[APPROVE]` item with the evidence and
   the exact prompt edit. Once the person approves, the CoS applies that
   edit itself (`manage_skills` targeted edit with `targetAssistantId`,
   allowed because the person owns the agent). It never makes the change
   unapproved, and it never delegates "update your skill" to the
   specialist.
7. **Autonomy ledger (full, optional).** Track each specialist's `[APPROVE]`
   items. A specialist earns a nomination to act without approval at ≥98%
   approved unchanged, over 4+ weeks and 20+ items. One bad autonomous
   action demotes it. A rule the person bypasses 3 weeks running is a wrong
   rule: recommend changing it. This is a **convention the CoS keeps**, not
   a product feature. Promotion means editing the specialist's skill (for
   example, taking it out of learning mode) on the person's approval.
8. **Visuals first for numbers (full).** For a numeric answer: the chart,
   then a short decisions table, then prose. Slack gets a bulleted
   scorecard. Charts need the CoS on Professional or above.

The CoS never edits records, never writes customer-facing content in a
specialist's lane, and is the only agent that schedules messages to the
person.

## Step 7: Deploy in order

Each step is previewed in `/implement`:

1. Specialist descriptions get the contract.
2. Team Brief pages: one per specialist, filed in one folder. The folder
   doesn't need workspace sharing; the person's own agents can read their
   pages without it. Share it only if teammates should read the briefs.
3. Specialist publish skills. Switch off their direct deliveries to the
   person.
4. The CoS description, with the generated routing table.
5. The CoS digest or briefing skill.

The CoS's briefing goes in after the sections it reads exist. Otherwise the
first briefing is all MISSING.

**Verify from runs:** after a week, `get_history` on every publish skill and
the briefing should show the work firing, substantive, and delivered. Only
the CoS should be delivering to the person.

## Never

- Design a CoS over another user's agents. Delegation is refused across
  owners.
- Have specialists delegate to each other inside a delegated task. A
  delegated run can't delegate.
- Hard-code skill or agent ids in a description. Route by name and slash
  command, and let the roster supply the ids.
- Promise fan-out from Slack threads.
- Add a dedicated CoS agent for someone with one or two agents. Promote one
  they already have.
