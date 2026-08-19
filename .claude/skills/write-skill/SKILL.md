---
name: write-skill
description: The teaching guide for authoring a single high-quality Day AI skill prompt. The thinking process, the structure, worked examples, Day AI language conventions, and the revision test. Read this before writing any skill. Usage: /write-skill (read as a guide) — or invoke to craft one prompt interactively.
---

# /write-skill

This is the craft document. A skill is a standing job you give a teammate's agent — work it does on a schedule, on an event, or on demand, without anyone having to ask. The difference between a skill that gets switched off in a week and one that becomes indispensable is almost entirely the prompt.

Read this before writing any prompt. If invoked directly with a target person and intent, walk through the process below interactively and produce one finished prompt.

The standard: would this be something you'd be proud to have the whole company see? If not, it's not done.

---

## Before you write: understand the person

You cannot write a good skill for a role you've reduced to a title. Answer these four questions first — from `workspace/PEOPLE.md`, the workspace graph, and the plan — and write nothing until you can:

1. **What is this person's job, *really*?** Not their title — their daily reality. A "VP of Sales" at a 12-person company is doing the selling. A "VP of Sales" at a 500-person company is coaching and forecasting. The same title, two completely different skills.

2. **What would be embarrassing for them to miss?** A stale deal that's about to slip. A promise they made on a call and forgot. A relationship going quiet. An email they never replied to. The best skills are built around *these specific moments*, not around generic "insights."

3. **What data does their agent actually have — and what's missing?** Don't just build around what exists. Notice what's absent and turn it into value: "I'd have prepped you for that call if their account were in your customer memory." Gaps become coaching, not silence.

4. **Where are they in their Day AI journey?** A teammate with a sparse, new workspace needs the skill to coach them toward value — the workspace-health content *is* the value early on. A teammate with a rich workspace needs pure intelligence and zero hand-holding. The prompt must know which one it's talking to.

---

## The structure of a great prompt

Once you understand the person, structure the prompt this way:

1. **Open with identity and context.** Tell the agent who it is, who the user is, what the company does, and what matters. Be specific — generic context produces generic output. *"You are Ada, Pici's agent. Pici runs 5–8 demos a week and lives in his calendar; he's the first human a prospect meets."* beats *"You are a helpful sales assistant."*

2. **Structure around what the person needs, not data sources.** This is the single highest-leverage move.
   - **Bad:** "Check email. Check the calendar. Check the pipeline."
   - **Good:** "Promises coming due. Relationships going quiet. Unfinished business from yesterday's calls. Prep for who he's meeting today."

3. **Name the patterns explicitly, with examples.** Not "surface relevant insights" — instead: *"A promise coming due: three weeks ago on the Meridian Analytics call he said he'd send pricing by end of month. That's tomorrow and no email has gone out."*

4. **Set the quality bar with a number.** "2–5 items, not 15. Each item should make them think, 'I'm glad someone caught that.'" Without a number, agents pad.

5. **Account for an empty workspace.** A prompt that assumes a fully instrumented workspace produces empty or misleading output for someone just starting. The best skills say what they *would* have caught if the data were there.

6. **Pull the human into the loop early.** On a skill's first runs, have the agent show its reasoning and invite correction: "I flagged these three as going quiet — were those the right ones?" As the skill matures, this fades.

7. **Specify anti-patterns.** What it must *not* do is as important as what it does: don't summarize meetings the person attended; don't repeat stale items; don't pad; don't manufacture urgency.

8. **Give explicit permission to say nothing.** "If there's genuinely nothing worth flagging, say so. Don't invent filler to look busy."

9. **End with delivery format.** Channel (Slack/email), shape (bullets/table/prose), and length. Match the channel: long analysis doesn't belong in a Slack DM.

---

## Worked example

**The ask:** "A morning skill for our AE, Sarah."

**Weak version (what not to write):**

> Every morning, check Sarah's email, calendar, and CRM. Summarize what's important and surface any relevant insights to help her have a productive day. Send it to Slack.

Why it fails: no identity, organized around data sources, "relevant insights" means nothing, no quality bar, no anti-patterns, would produce identical output for any salesperson on earth.

**Strong version:**

> You are Sarah's agent. Sarah is an AE at Acme selling to mid-market RevOps leaders; she runs ~6 active deals at a time and her biggest risk is letting a warm deal go cold while she's heads-down on the next call. Every weekday at 7:30am, before she opens her laptop, give her the one screen she needs.
>
> Lead with the **single most important thing** — name the person, the company, the dollar figure, the specific risk. Then, only if they earn the space:
> - **Promises coming due:** something she committed to on a call that's now due or overdue. Quote what she said and when.
> - **Going quiet:** an open deal with no touch in 7+ days where the last signal was positive. Draft the re-engagement email so she can send it in one click.
> - **Today's prep:** for each external meeting today, the one thing she needs to remember — last conversation, open question, who else is in the room.
>
> 2–4 items, never more. If a deal is genuinely quiet because it's dead, don't resurface it. Don't summarize meetings she ran. Don't tell her things she already knows. If it's a quiet morning, say so in one line and stop.
>
> Deliver as a Slack DM: bold the names, one line per item, the draft email in a collapsed block.

Notice: it would produce *different* output for a different person, every section earns its place, and you could not delete the context paragraph and still know who it's for.

---

## Skills that lean on a living guide

Some skills should follow a shared playbook page — a discovery guide, a demo guide, an objection playbook. **Reference the page; never paste it:**

- Name the page by title and objectId in the prompt, and instruct the agent to **read it at run time** before doing the work: *"Read the Discovery Guide page and prep against its current questions."*
- Never embed the guide's content in the prompt. The whole point of the flywheel is that the guide improves continuously; a pasted copy freezes it and drifts from the shared version.
- Producer-side skills (the Playbook Editor pattern) review recent calls **against the current guide** and propose targeted page edits with the evidence — the specific call moments that justify the change — never a wholesale rewrite.

## Day AI language conventions

Every line a teammate reads from a skill must use Day AI's language:

- The AI is **"your agent"** or its name — never "your assistant" or "the bot."
- The data layer is **"your customer memory"** or **"what I know"** — never "the CRM" or "the database."
- Frame value as **what becomes possible**, not time saved.
- Name capabilities by what they do: *draft an email*, *search the pipeline*, *send a Slack message*, *update a contact*.
- No category labels, no competitor comparisons, no leading with "AI."

---

## The revision test

After drafting, before deploying, check every one:

1. **Would this produce different output for different people?** If swapping the target person gives the same briefing, it's not specific enough.
2. **Does every section earn its place?** Cut any section that would rarely produce something useful.
3. **Is there a single line that says "surface insights" or "provide relevant information"?** Delete it. Replace with a named pattern.
4. **Could you delete the identity/context paragraph and still know who this is for?** If yes, it isn't grounded enough.
5. **If the workspace is half-set-up, does the skill handle that gracefully?** Or does it produce empty output?
6. **Did you assume any reporting relationships?** Never infer who manages whom from titles — only state relationships confirmed in the data.
7. **Is the quality bar a number?** "2–5 items" not "a few."
8. **Does it have permission to say nothing?** And explicit anti-patterns?
9. **Is the channel/format/length specified and matched to the channel?**
10. **Does every line obey the Day AI language conventions?**

---

## Config that ships with the prompt

When you hand a finished prompt to `agent-implementor` (or write the `manage_skills` call yourself), specify:

- **name** and **slashCommand** (unique per agent, lowercase-hyphens).
- **triggerType**: `SCHEDULE` (cron `triggerValue` + `timezone`; the platform accepts exactly four shapes, minutes on :00/:15/:30/:45 — daily `0 8 * * *`, weekdays `0 8 * * 1-5`, weekly `0 8 * * 1`, every 4 hours `0 */4 * * *`; a monthly or twice-weekly cron is rejected), `EVENT` (one of the platform's closed list of seven event types — see `docs/INSTRUCTION_ARCHITECTURE.md`), or `NEITHER` (on-demand).
- **notificationType**: `["slack"]`, `["email"]`, or both — and `slackNotificationChannels` if Slack. **Always set it explicitly on a scheduled or event skill: with no notification target the skill runs and delivers nowhere** (the run records `notification: null`). Choose the channel to match the output (long analysis → email; a short nudge → Slack), and set Slack explicitly when that's where the person actually works.
- **scope**: a per-agent skill (`targetScope: "agent"` + the teammate's `targetAssistantId`) when the prompt is tailored to one person; a workspace-library skill (`targetScope: "workspace_library"`) when a whole team needs the same capability. A **Managed** library skill encodes as `deploymentMode: "SHARED"`, a **Template** as `"TEMPLATE"`; the API never accepts `"MANAGED"` as a value.
- Remember the **tier budget**: automated skills consume the target agent's slots. One excellent scheduled skill beats three thin ones.

A morning skill defaults well at `0 8 * * 1-5` with the person's timezone. Crons are local to the `timezone` you set, so `0 8` means 8am *their* time — never write a UTC-offset cron.
