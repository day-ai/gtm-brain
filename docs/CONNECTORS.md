# What Day AI Connects To

> **What this is:** the connector and import reference. `/discover` reads it during
> Gate 1 (tech stack) and when ranking first-win candidates, so "does Day AI
> connect to X?" becomes a decision on the spot instead of a parked blocker.
>
> **The catalog is live; this file is the map to it.** This file teaches the
> shape of the answer (the kinds of connections that exist and how discovery
> uses them). The current list of what actually connects lives in two places,
> and those always win over anything printed here:
>
> 1. **https://www.day.ai/resources/integrations-connectors**: Day AI's
>    maintained reference for workspace integrations and the major agent
>    connectors.
> 2. **The product itself** (connected workspaces only): Settings →
>    Integrations for workspace-level connections, and the connector picker on
>    any agent for the current MCP connector catalog.
>
> When this file and a live source disagree, the live source is right; update
> this file. If neither live source answers the question, record it as an open
> question for your Day AI contact; never treat an absence here as proof.

**Last verified:** 2026-11-18 (against the Day AI product source; re-check day.ai/resources/integrations-connectors and the in-product picker before relying on a beta or newly listed connector)

---

## The two kinds of connections

Day AI draws a hard line between **integrations** (background data sync into
your customer memory) and **MCP connectors** (live tools your agents can use).
Discovery needs the distinction constantly: an integration answers "will the
evidence be *in* the workspace?"; a connector answers "can the agent *act on*
or *read from* an external system at run time?" A connector does not
bulk-import history, and an integration does not give an agent actions.

## Integrations (data flows into your customer memory)

Workspace integrations live under Settings → Integrations and are set up by
the workspace **Owner** (Admins can't by default), so name the Owner as the
step's owner in the rollout plan. Email and calendar are each person's own
connection.

| Source | Level | What syncs |
|--------|-------|-----------|
| **Gmail** | Per-user | Emails, thread summaries, contact extraction. Per-user sharing rules control what teammates see (see `workspace/PRIVACY.md`). |
| **Google Calendar** | Per-user | Calendar events and attendees. One Google OAuth per user covers both. |
| **Day AI notetaker** | Per-user | Meeting recordings, transcripts, action items. Rides the calendar connection; per-user auto-record toggles for internal and external meetings. |
| **Slack** | Workspace | Messages and users from connected channels; channels are added individually or in bulk. |
| **Gong** | Workspace | Meeting recordings, transcripts, participants. One-way import. Confirm the sync cadence for new calls with your Day AI contact before promising it in a rollout plan. |
| **Granola** | Workspace | Meeting notes and transcripts, via an Enterprise API key. |
| **Fireflies** | Workspace | Meeting recordings and transcripts from Fireflies. Confirm what history imports with your Day AI contact. |
| **Linear (customer sync)** | Workspace | The other direction: pushes Day AI organizations out to Linear as customers, so product and engineering see who's asking. Distinct from the Linear MCP connector below. |
| **Zapier** | Workspace | Pushes person and organization records from other apps; good for long-tail systems with no direct connector. Configured on Zapier's side. |

## MCP connectors (your agents get live tools, not a data sync)

Inside Day AI, each person connects MCP connectors for their own agents
(User Settings → Connectors), with per-tool permissions (always allow / needs
approval / never). These give an agent live reads and actions in the external
system. **External connector tools ask for approval by default**; a skill
that names a connector (`mcpProviders`) has it pre-approved for that skill's
runs, using the skill owner's own connection, never a shared credential.

The majors, per the live reference page: **Linear** (create and manage
issues), **Notion** (search and manage pages and databases), **HubSpot** (CRM
objects, engagements, pipelines), and **Affinity** (CRM people,
organizations, lists, notes). The in-product catalog is larger and changes
over time; the connector picker is the authoritative list, so check it (or
ask your Day AI contact) for any system not named on the page.

As of the last verification the catalog also listed, among others:

- **CRM and data:** Salesforce, Attio, BigQuery (beta)
- **Prospecting and outbound:** Apollo, Clay, lemlist, HeyReach, Nooks,
  Common Room, Sumble, Harmonic, Crossbeam (beta)
- **Product analytics:** Amplitude, PostHog
- **Docs and contracts:** Guru, PandaDoc, Docusign, Asana
- **Support and billing:** Intercom, Tabs
- **Automation:** n8n
- **Microsoft 365:** Outlook 365 Mail, Outlook 365 Calendar, Microsoft
  Teams (all beta)

All of these are live tools for an agent, not data syncs. Treat beta
connectors as "confirm before you promise" in a rollout plan.

Teams can also register their own custom MCP servers in workspace settings.

Setup nuances worth surfacing in Gate 1, because they create small
IT-adjacent steps that need a named owner:

- **HubSpot, Salesforce, and BigQuery** require the customer to register an
  OAuth client in their own account and enter its credentials during setup.
- **Affinity** is API-key based and needs a workspace admin to set it up.
- For anything else, confirm the setup mechanic in the product before telling
  a customer it is "connect and authorize"; don't assume.

One harness-level note: the Day AI MCP server this repo uses does not
expose a tool that lists or connects connectors (that's chat or the UI
only), so verification is always the live page plus the in-product surfaces
above, never an MCP call. Connecting is each person's own step: put it in
their ENABLEMENT setup, not in `/implement`.

## Imports (bulk history in)

- **CSV import** with column mapping, error reporting, and progress tracking.
  Contacts confirmed; the import tooling is object-type based, so plan the
  mapping per object in `rollouts/preflight/IMPORT_PLAN.md`.
- The standard bridge pattern for an incumbent CRM (Customer Memory posture):
  one-time CSV export/import for history, plus an MCP connector if the agent
  needs live reads of the incumbent.

## Day AI's own MCP server (the other direction)

External Claude clients (Claude Code, Claude Desktop, any agent framework)
connect to Day AI at `day.ai/api/mcp`. That is what this harness uses, and it
is how the preflight payload gets applied on connect day.

## Not natively synced today

- **Microsoft 365 / Outlook / Teams:** no native account *sync*; the
  account model is Google, and campaigns send from Gmail mailboxes. Outlook
  365 Mail, Calendar, and Teams exist as **beta MCP connectors**: an agent
  can read and act live, but the mail doesn't flow into customer memory the
  way Gmail does. A Microsoft-stack prospect should flag this to their Day
  AI contact before scoping a pilot around email evidence or campaigns.
- **Salesforce / HubSpot live two-way sync:** not a native sync (both are MCP connectors). The supported
  pattern is the CSV bridge plus an MCP connector for live reads. Do not
  design a first win that assumes bidirectional CRM sync.

## How discovery uses this catalog

- Gate 1: set each system's disposition (stays / syncs / replaced) from the
  live sources above, not from guesses. A system in the "not natively synced"
  list with no workable bridge is an integration blocker; name it and assign
  an owner.
- First-win filter 2 ("not integration-gated"): a candidate is only gated if
  the data it needs has no integration or workable bridge. Gong-evidence and
  Granola-evidence candidates are not gated; they import.
- If a system is not covered by either live source, record it as an open
  question for the operator's Day AI contact and mark dependent skills
  `GROUND-AFTER-CONNECT`; never silently assume a connector exists.
