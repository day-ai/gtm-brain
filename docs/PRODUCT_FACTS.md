# Day AI Product Facts

> **What this is:** the brain's reference for what the Day AI product can do
> and what each capability is gated on: agent tiers, automated-skill slots,
> campaigns, prospecting, AI-populated properties, live dashboards, the skill
> library, and the Day AI MCP surface this harness drives. Skills and agents
> in this repo cite this file instead of restating facts, so a product change
> is a one-file edit.
>
> **The live sources win.** For prices, always fetch https://day.ai/pricing
> at recommendation time (CLAUDE.md's pricing rules); connected, the
> workspace's own billing and tier data is authoritative. For tools, the
> connected MCP tool list is authoritative: if a tool named here is missing
> or has changed shape, trust the tool list, say so, and update this file.
> Never assert a figure from this file that a live source contradicts.

For *how* to run a use case well (best practice, not limits), link the
matching Day AI guide from [`GUIDES.md`](GUIDES.md).

**Last verified:** 2026-11-18, against the Day AI product source and
https://day.ai/pricing.

---

## Billing model: per agent, not per seat

- **Day AI bills per agent.** Each agent is billed at its tier's flat monthly
  rate. There are no per-seat fees and no usage-based pricing. Human
  teammates join free once the workspace has at least one Professional (or
  higher) agent.
- **The bill is agents × tier price.** Volume discounts apply automatically
  as the agent count grows, and annual billing is discounted. Read the
  current tiers, discounts, and cadence from https://day.ai/pricing (and
  say which cadence a figure reflects).
- **Free members** can search, view, and share, and their meetings and
  email are captured. They have no agent and run no skills.
- So the cost of a recommendation is always **which agents, at which tier**:
  never "a seat." An invite on its own costs nothing.

## Agent tiers

There are four customer tiers, ranked cheapest to most capable, plus Free
(no paid agent). Each tier includes everything in the tiers below it.

| Tier | What it adds | Automated-skill slots | Schedules allowed |
|------|--------------|-----------------------|-------------------|
| **Free** | Capture, search, view, share; no agent tools | 0 | none |
| **Turbo** | Task management, email drafts, web search, Pages and notifications | 2 | daily, weekdays, weekly |
| **Professional** | Pipeline and opportunity management, custom properties and views, pipeline analytics, analytics and code execution, **live dashboard widgets**, working campaigns (drafting, approving, sending); can create a workspace | 5 | daily, weekdays, weekly |
| **Executive** | **Creating and launching campaigns**, custom campaign instructions, contact identification and external people/organization search | 10 | adds every 4 hours |
| **Super** | **Lead Finder** (agentic web prospecting, lookalikes of your best customers), campaigns that prospect their own leads, multiplayer campaigns, and **up to 20 AI-populated custom properties** per agent | 10 | adds every 4 hours |

- **Super is sold on annual plans only**, sales-led ("talk to sales"). It
  isn't offered in self-serve onboarding, which offers only Professional and
  Executive.
- **Partner** is a staff-only $0 tier that Day AI deploys for Partner
  Program members: chat only, no automated skills, no premium tools. It
  never appears in customer tier pickers. Never recommend it.
- The tier gates **tools**. An agent's tools come from its tier's tool list,
  so a skill that needs a tool the target agent's tier lacks simply can't
  call it, whatever the prompt says.

### Automated-skill slots, precisely

- A skill uses a slot when it is **enabled** and has a **SCHEDULE or EVENT**
  trigger. On-demand skills (`NEITHER`) are free. Default and system skills
  don't count; **Day AI library templates do**, once automated.
- The limit is checked against the **target agent's** tier when an
  automated skill is created or assigned (including library deploys and
  campaign provisioning).
- **On a tier downgrade, skills over the new limit, or on a schedule the new
  tier doesn't allow (every 4 hours), are switched off.** Read the agent's
  skills before recommending a downgrade.
- **Each campaign uses two of its agent's automated slots** (the drafting
  skill and the status skill). A Professional agent with five slots that
  works two campaigns has one slot left.

### Who needs which tier (the gates the product enforces)

| To… | Needs | Checked against |
|-----|-------|-----------------|
| Run any scheduled or event skill | Turbo or higher | the target agent |
| Build live dashboard widgets | Professional or higher | the agent doing the build |
| View, work, approve, and send a campaign's drafts | Professional | the user's highest-tier agent |
| Be the agent that works a campaign | Professional | that agent |
| Create, set up, or launch a campaign | Executive | the user's highest-tier agent |
| Search external people and organizations | Executive | the agent |
| Prospect net-new accounts (Lead Finder) | Super | the user's highest-tier agent |
| Turn on AI population (`aiManaged` / `useWeb`) for a custom property | **a Super agent that owns the property, with a free slot (max 20)** | the owning agent |

The AI-property gate has one exception: a small set of workspaces that were
already using AI-managed properties when Super launched keep that ability on
any agent, as long as they are billed annually. Don't assume a workspace is
in that set; if a property create is refused, read the error.

See [`playbooks/choose-a-tier.md`](playbooks/choose-a-tier.md) for how to
turn these gates into a recommendation.

## Campaigns (generally available)

A campaign is an agent-worked outbound sequence: an audience is enrolled
(from a CSV, from customer memory, or from prospecting on Super), the agent
drafts each touch, a human approves the drafts, and the drafts send from a
teammate's Gmail mailbox. Follow-ups stop when someone replies, unsubscribes,
or bounces.

- **Sequence:** up to 20 steps across email, call, and LinkedIn, each with a
  delay. Default sequencing is 3 touches (2–5) at least 3 days apart.
- **Review is mandatory today:** every draft lands in a review queue and is
  approved one by one or in bulk ("Send all approved"). Auto-send trust
  levels exist in the schema but aren't enforced yet; don't promise them.
- **Sending mailbox:** one of the creator's connected Gmail accounts. It
  locks after the first drafting run, and follow-ups stay on the first
  touch's thread.
- **Send limits:** each mailbox sends at most **50 emails per rolling 24
  hours by default**. A workspace Owner can raise this to a hard maximum of
  **250** (Settings → Sending) after accepting a sender-reputation warning.
  The limit counts every campaign and one-off send from that mailbox.
- **Cost:** see the tier gates above, plus **two automated slots per
  campaign** on the agent that works it.
- **Via the MCP:** the harness can read campaigns (`list_campaigns`,
  `get_campaign_details`, `get_campaign_drafts_in_flight`) and manage them
  (`create_or_update_campaign`, `add_campaign_members`, `populate_campaign`,
  `draft_campaign_outreach`, `refine_campaign_drafting`, …). **Guided setup
  and launch run in Day AI itself**, not over the MCP.

See [`playbooks/set-up-a-campaign.md`](playbooks/set-up-a-campaign.md).

## Live dashboards on Pages

A Day AI Page can be a **live dashboard**: widgets store a query, not data,
and recompute every time the page loads, so relative windows like "last 30
days" slide automatically and the page never needs a schedule.

- **`create_widget`** builds a chart from customer memory. It can count, sum,
  average, take a ratio (e.g. win rate), make a stage-history funnel, or give
  a weighted forecast. It can group by stage, owner, date, relative date,
  relationship, or campaign member status. It can add up to three viewer
  filter dropdowns. The tool picks the chart (KPI tiles and gauges, bars,
  lines, funnels, stage conversion, heatmaps, Sankey, …) from the question
  you put in `chart.title`.
- **`create_table_widget`** builds a live table: a flat list, a grouped list,
  or a pivot, up to 1000 rows. Cells can be formatted as meters, heat,
  badges, or stacks. A table is the appendix of a dashboard, not its lead.
- **Report queries.** Both tools accept a `query` (instead of
  `source`/`measure`) for history and movement:
  - `asOf` snapshots of the pipeline: one point in time (`{at}`) or a series
    (`{every: week|month|quarter}`). These have the fields `createdInPeriod`,
    `wonInPeriod`, and `closesInPeriod`.
  - **`native_opportunity.changes`**: one row per deal movement in a window
    (New, Advanced, Close date pushed, Slipped, Increased, Decreased, Won,
    Lost).
  - **`native_opportunity.bridge`**: start-to-end pipeline walk; the signed
    steps sum exactly.
  - property and stage history, and an `activity` dataset (email, events,
    meetings, calls, tasks).
  - calculations: running totals, rank, percent of total, period-over-period
    change, and moving averages.
- **Layout markup** (in `create_page` / `update_page` HTML):
  - `<day-ai-dashboardgrid data-columns="2|3">`: 1–3 widgets, tables, or
    panels per row. Use 3 columns only for compact KPI tiles.
  - `<day-ai-panel variant="card|callout|fold">`: a card panel can carry a
    headline `metric`.
  - `<day-ai-meter>` for progress, and `<day-ai-chip>` for inline status.
  - Static `<day-ai-chart>` is a snapshot. **Never use it on a dashboard.**
- **Embedding from this harness (MCP):** a widget tool returns
  `{recipe, widgetRef}`. Inside Day AI chat, pages embed the short
  `recipe-ref`. **An MCP call has no chat thread, so `recipe-ref` cannot
  resolve here.** Embed the full `recipe="…"` string exactly as returned:
  never retype or edit it.
- **Gates:** Professional or higher for the widget tools. Who can see the
  numbers follows the page's sharing, and each viewer's query runs with
  their own access.
- Custom-property ids used in a widget must be copied verbatim from
  `read_crm_schema`; a wrong id is rejected rather than charting $0.

See [`playbooks/build-a-dashboard.md`](playbooks/build-a-dashboard.md).

## Pages, sharing, and folders

- `create_page` makes a page **private by default.** Share it with the whole
  workspace by passing `publishedForUserAt` (an ISO timestamp) on create or
  update, or file it in a folder created with `shareWithWorkspace: true`.
  `""` unshares it. `madeExternalAt` makes a public link: an outward-facing
  act, so always confirm it with the operator.
- Page HTML over ~8,000 characters goes through `pageHtmlFileId`.
- `get_share_url` returns a page's link. `read_folder` and `move_to_folder`
  manage filing.

## Skills: what the platform supports now

- **Triggers:** `SCHEDULE` (daily, weekdays, weekly, every 4 hours; 15-minute
  alignment; every 4 hours needs Executive or higher), `EVENT` (seven
  selectable events, listed in `INSTRUCTION_ARCHITECTURE.md`), or `NEITHER`.
- **Attachments:** a skill can attach up to **10 Pages or Folders**, each
  with an instruction of up to 500 characters (`attachmentChanges`). The run
  reads them live (a folder includes pages added later). This is the
  preferred way to bind a skill to a living guide.
- **Includes:** a skill can fold in up to 5 other skills on the same agent
  (`includedSkills`), so a shared voice ("Email Writing") lives in one place.
- **Connectors:** a skill can use the owner's connected MCP connectors
  (`mcpProviders`). Connectors named on a skill are pre-approved for its
  runs.
- **Targeted edits and versions:** `manage_skills update` takes
  `promptEdits` (`oldPromptMatch` → `newPromptReplace`) with
  `basedOnPromptVersion`. Prefer this over full replace. `list_versions` and
  `get_version` read a skill's edit history, and restoring an old version is
  `get_version` and then `update`.
- **Run history:** `get_history` returns run digests with the `notification`
  delivery block, and can filter to runs that called a given tool.

## The skill library

Two kinds of library skill exist:

1. **Your workspace library:** skills an Admin creates with
   `targetScope: "workspace_library"`, as **Managed** (`deploymentMode:
   "SHARED"`; synced to every assigned agent) or **Template** (`"TEMPLATE"`;
   each agent gets an independent fork). Admins push these to agents with
   `manage_skills → deploy` / `undeploy`.
2. **Day AI's library templates:** code-defined skills every workspace sees
   in the Skill Library. They aren't editable source records and **can't be
   pushed with `manage_skills deploy`.** Someone adds them per agent in Day
   AI (**Add to agent**). For a customised team version, create a
   workspace-library copy.

Day AI's GTM templates (18, available in every workspace):

| Template | Slash | Minimum tier |
|----------|-------|--------------|
| Account News | `/account-news` | Turbo |
| Anonymize for Sharing | `/anonymize` | Professional |
| Buying Group | `/buying-group` | Executive |
| CRM Cleanup | `/crm-cleanup` | Professional |
| Customer Health | `/customer-health` | Turbo |
| Deal Brief | `/deal-brief` | Turbo |
| Deal Follow-Up | `/deal-follow-up` | Turbo |
| Deal Scouting | `/deal-scouting` | Turbo |
| Enrich a List | `/enrich-list` | Executive |
| Follow-Up Sweep | `/follow-up-sweep` | Turbo |
| Meeting Prep | `/meeting-prep` | Turbo |
| Phone Capture | `/phone-capture` | Turbo |
| Pipeline Dashboard | `/pipeline-dashboard` | Professional |
| Pipeline Review | `/pipeline-review` | Turbo |
| Proof Points | `/proof-points` | Turbo |
| Record Check | `/record-check` | Professional |
| Save Plan | `/save-plan` | Turbo |
| What We Heard | `/what-we-heard` | Turbo |

Older templates (Actions, Opportunity Automation, Call Coaching, Deal
Coaching, and others) also live there. Account Map and the enablement skills
are Day AI–internal and don't appear in customer workspaces. See
[`playbooks/deploy-library-skills.md`](playbooks/deploy-library-skills.md).

## The Day AI MCP surface (what this harness can and can't do)

Available to an authorized Owner/Admin through `https://day.ai/api/mcp`. The
exact set depends on the authorizing agent's tier.

- **Admin:**
  - `assistant_settings` (read / update / list)
  - `manage_skills` (list, get, create, update, delete, reset_prompt,
    get_history, list_versions, get_version, deploy, undeploy)
  - `manage_workspace_members` (list_configuration, invite_member,
    resend_invite, revoke_invite, update_invite_role, **update_member_role**,
    **remove_member**, enable/disable_auto_invite, list_suggested_invites,
    navigate_to_billing)
  - `manage_workspace_instructions` (list_configuration, update; **7500
    characters**)
- **Content:** `create_page`, `update_page`, `read_page`,
  `create_or_update_folder`, `read_folder`, `move_to_folder`,
  `get_share_url`, `create_widget`, `create_table_widget`, `create_chart`.
- **Schema and data:** `read_crm_schema`, `create_or_update_custom_property`,
  `backfill_custom_property`, `search_objects`, `analyze_pipeline_metrics`,
  `create_view`, `create_or_update_list`, the import tools.
- **Campaigns:** see above.

**What the public MCP can't do** (so the harness must route to a human in
Day AI):

- **Create an agent.** This is UI-only and billing-gated. Its tier can't be
  changed over the MCP either.
- **Connect or list MCP connectors.** That's chat or the UI only.
- **Edit workspace rules.**
- **Email customers directly.** Skills deliver to their owner;
  `send_notification_mcp` notifies the authorizing user (email or Slack DM)
  or posts to a Slack channel.
- **Launch a campaign.**
- **Add a Day AI library template to an agent.**

Some tools you may see referenced elsewhere (`get_pipeline_definition`, a
page-sharing audit, workspace integration and connector lists) belong to
Day AI's **staff-only** admin endpoint, not the customer MCP. Don't design
around them. To read a pipeline's stages, use `read_crm_schema` and
`search_objects` on `native_pipeline` / `native_stage`.
