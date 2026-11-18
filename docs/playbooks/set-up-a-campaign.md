# Playbook: set up a campaign

> Campaigns are generally available. The capability and its gates are in
> [`../PRODUCT_FACTS.md`](../PRODUCT_FACTS.md#campaigns-generally-available).
> This playbook is how the brain plans one so that launch day is a click,
> not a project. Campaigns send email to real people from a teammate's
> mailbox: the most outward-facing thing the harness touches. Preview
> everything; approve nothing on the operator's behalf.

## When a campaign is the right answer

Use a campaign when an outcome is **"reach a defined audience with a
multi-touch sequence, personalised from what we know"**: an event invite, a
re-engagement of closed-lost deals, an expansion motion into a customer
segment, or outbound to a target-account list. It is not the answer for:

- one-to-one follow-up after a meeting (that's a Coach or Follow-Up Drafter
  skill);
- a single announcement with no sequence (a draft and a send);
- anything the team won't staff a human reviewer for. Every draft is
  approved by a person today.

## The decisions to record (in the initiative or outcome it serves)

1. **Goal and success measure.** Use numbers: meetings booked, replies,
   registrations, with a baseline and a by-date. The campaign's own member
   statuses become the dashboard (see below).
2. **Audience source.** One of:
   - a CSV list;
   - a segment of customer memory (a list, view, or `search_objects` query);
   - **prospecting**, which needs a **Super** agent (Lead Finder;
     annual-only).
   Name who vouches for the list's quality.
3. **Owner and agents:**
   - The person who **creates and launches** needs an **Executive or Super**
     agent.
   - The agent that **works** the campaign needs **Professional or higher**,
     and **two free automated slots**.
   - Teammates working their own slice (multiplayer) need Professional;
     multiplayer campaigns themselves need Super.
   Check slots before promising anything: `manage_skills → list` on the
   working agent.
4. **Sending mailbox.** One of the creator's connected Gmail accounts. It
   locks after the first drafting run. Microsoft-mail teams can't send
   campaigns from their own mailbox natively; flag this at Gate 1.
5. **Volume vs. send limits.**
   - Each mailbox sends **50 a day by default**, up to **250** if a
     workspace Owner raises it after the deliverability warning.
   - The limit is shared with every other campaign and send from that
     mailbox.
   - Do the arithmetic for the operator: audience × touches ÷ daily limit =
     days to complete. A 600-person, 3-touch campaign at 50/day is about 36
     sending days.
6. **Sequence.** Touch count (default 3, max 5 per member under the
   sequencing settings; up to 20 defined steps), spacing (default 3 days),
   channels (email, call, LinkedIn), and the send window.
7. **Voice and guardrails.** The drafting follows the working agent's
   **Email Writing** skill and any campaign-specific instructions
   (Executive+). Write those to the `/write-skill` bar: who it's from, what
   it's for, what never to say. If the team has a messaging page, attach it.
8. **Reviewer and cadence.** Who approves drafts, and how often they clear
   the queue. An unreviewed queue is a stalled campaign.

## What the harness does vs. what happens in Day AI

| Step | Where |
|------|-------|
| Plan the decisions above, and draft the drafting instructions | This repo (`initiatives/`, `rollouts/<date>-campaign-<slug>/CAMPAIGN.md`) |
| Check tiers and slots on the creator's and workers' agents | MCP (`assistant_settings → list`, `manage_skills → list`) |
| Create the campaign record, add members, refine drafting | MCP (`create_or_update_campaign`, `add_campaign_members` / `populate_campaign`, `refine_campaign_drafting`), on approval |
| Guided setup, choosing the sending mailbox, launch | **Day AI**, by the creator, since launch isn't on the MCP |
| Review and approve drafts | **Day AI** review queue, by the named reviewer |
| Track results | A live dashboard page (below), plus `get_campaign_details` |

## Track it

Build a small dashboard page per the dashboard playbook:
- `create_widget` on `native_contact` members of the campaign, grouped by
  campaign member status;
- a ratio widget for reply or meeting rate;
- a table of members awaiting a next touch.
Attach the page to the owner's weekly review skill, if there is one.

## Cost line (for the preview)

Show:
- the agent tiers involved (creator Executive or Super, worker Professional
  or higher) and whether any is an upgrade;
- **two automated slots per campaign** on the working agent, and what that
  leaves;
- whether prospecting is in scope (Super, annual).
Price per CLAUDE.md's pricing rules.
