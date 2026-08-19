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

**Last verified:** 2026-08-17 (against day.ai/resources/integrations-connectors and the live product surface)

---

## The two kinds of connections

Day AI draws a hard line between **integrations** (background data sync into
your customer memory) and **MCP connectors** (live tools your agents can use).
Discovery needs the distinction constantly: an integration answers "will the
evidence be *in* the workspace?"; a connector answers "can the agent *act on*
or *read from* an external system at run time?" A connector does not
bulk-import history, and an integration does not give an agent actions.

## Integrations (data flows into your customer memory)

| Source | Level | What syncs |
|--------|-------|-----------|
| **Gmail** | Per-user | Emails, thread summaries, contact extraction. Per-user sharing rules control what teammates see (see `workspace/PRIVACY.md`). |
| **Google Calendar** | Per-user | Calendar events and attendees. One Google OAuth per user covers both. |
| **Day AI notetaker** | Per-user | Meeting recordings, transcripts, action items. Rides the calendar connection; per-user auto-record toggles for internal and external meetings. |
| **Slack** | Workspace | Messages and users from connected channels; channels are added individually or in bulk. |
| **Gong** | Workspace | Meeting recordings, transcripts, participants. One-way import. Confirm the sync cadence for new calls with your Day AI contact before promising it in a rollout plan. |
| **Granola** | Workspace | Meeting notes and transcripts, via an Enterprise API key. |
| **Zapier** | Workspace | Pushes person and organization records from other apps; good for long-tail systems with no direct connector. Configured on Zapier's side. |

## MCP connectors (your agents get live tools, not a data sync)

Inside Day AI, agents can be granted MCP connectors with per-tool permissions
(always allow / needs approval / never). These give an agent live reads and
actions in the external system.

The majors, per the live reference page: **Linear** (create and manage
issues), **Notion** (search and manage pages and databases), **HubSpot** (CRM
objects, engagements, pipelines), and **Affinity** (CRM people,
organizations, lists, notes). The in-product catalog is larger and changes
over time; the connector picker on any agent is the authoritative list, so
check it (or ask your Day AI contact) for any system not named on the page.

Setup nuances worth surfacing in Gate 1, because they create small
IT-adjacent steps that need a named owner:

- **HubSpot, Salesforce, and BigQuery** require the customer to register an
  OAuth client in their own account and enter its credentials during setup.
- **Affinity** is API-key based and needs a workspace admin to set it up.
- For anything else, confirm the setup mechanic in the product before telling
  a customer it is "connect and authorize"; don't assume.

One harness-level note: the Day AI MCP server this repo uses does not
currently expose a tool that lists the connector catalog, so verification is
always the live page plus the in-product surfaces above, never an MCP call.

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

- **Microsoft 365 / Outlook / Teams:** no native account connection; the
  account model is Google. A Microsoft-stack prospect should flag this to
  their Day AI contact before scoping a pilot around email evidence.
- **Salesforce / HubSpot live two-way sync:** not a native sync. The supported
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
