# MCP Requirements — gaps the agent-value analyst needs closed

The `data-analyst` agent and `/agent-audit` can already evaluate a lot from the existing Day AI MCP (membership, agent coverage, skill prompt craft, agent identity). But several high-value judgments — above all, *"is this agent actually delivering value?"* — can't be made from configuration alone. They need data the **admin** MCP exposes internally but the **public** `day.ai/api/mcp` server does not yet.

This doc records those gaps as paste-ready Linear tickets. Until they ship, the analyst is explicit in its reports about what it can confirm (build quality) vs. what it can't (realized value), and cites this file.

> **Scope note.** The DAY-2585 "Admin/Owner MCP Tools" items (`assistant_settings` `list` + `targetAssistantId`, `manage_skills` cross-agent, `list_suggested_invites`, `resend_invite`) are treated as **live** in this harness. The items below are **net-new** beyond DAY-2585.

**Last updated:** 2026-06-07

---

## Priority order

| # | Capability | Unblocks | Priority |
|---|------------|----------|----------|
| 1 | Skill run history (read) | Dimension 3 — is a skill *delivering*, not just configured | **P0** |
| 2 | Engagement metrics (workspace + per-agent/user) | Dimension 3 value + Dimension 1 activation | **P0** |
| 3 | Member activity / last-active | Dimension 1 — dormant vs active, who to nudge | P1 |
| 4 | Pipeline & stage definitions / CRM schema (read) | Dimension 2 — process-fluent Coach agents | P1 |
| 5 | Object completeness / `updatedAt` in search results | Dimension 3 — measuring CRM hygiene at scale | P2 |

---

## 1 — Skill run history (read) · P0

**Title:** Public MCP: read skill run history (recent run output + delivery result)

**Problem.** `/agent-audit` can read a skill's prompt and trigger config, but cannot see whether it actually fires, what it produces, or whether the output is substantive vs. hollow. "This skill has a daily schedule" is not evidence it delivers value — a great prompt over empty data produces nothing useful, and a stale skill keeps a schedule long after it stopped mattering. Today the analyst must caveat every effectiveness claim.

**What we need.** A read tool (parallel to the internal admin `get_skill_history`) that, given a skill (and `targetAssistantId` for Admin/Owner), returns the last N runs: timestamp, whether it fired on schedule, the run's output/transcript (or a substantive summary), and the **delivery result** of any notification call (delivered via Slack DM / email / failed) — not just the channel config.

**Acceptance criteria.**
- Returns ≥ the last 5 runs for a given skill, with timestamps and output.
- Surfaces the notification *result* (delivered/failed + channel), so delivery is confirmed from the event, not inferred from `slackNotificationChannels`.
- Admin/Owner can read any agent's skill history via `targetAssistantId`; a user can read their own.
- Handles "never run" cleanly (empty, not error).

**Notes.** This is the single highest-value gap. With it, `/agent-audit` can score each skill Thriving / Producing-but-hollow / Not-firing on evidence.

---

## 2 — Engagement metrics · P0

**Title:** Public MCP: workspace + per-agent/user engagement metrics

**Problem.** The analyst can't distinguish an agent a human actively chats with from a provisioned-but-dormant one, and can't measure whether scheduled skills drive any engagement. This blocks both the value verdict (Dimension 3) and the activation analysis (Dimension 1 — who's a real user vs. a cold seat).

**What we need.** A metrics tool (parallel to the internal admin `get_engagement_metrics`) returning, for a window (e.g. last 30d), at the workspace and per-agent/user grain: human-initiated chat threads (`manualThreads`), automated/skill threads (`asyncThreads`), skills enabled, and active-user count.

**Acceptance criteria.**
- Workspace-level totals and a per-agent (or per-user) breakdown for a configurable window.
- `manualThreads > 0` is reliably the "a human is actively using this" signal.
- Admin/Owner sees all agents; a user sees their own.

---

## 3 — Member activity / last-active · P1

**Title:** Public MCP: per-member activity / last-active signal

**Problem.** `list_configuration` returns who's a member and their role, but not whether they're *active*. Dimension 1 needs to separate active members, dormant members, and never-activated seats to know who to nudge and who to re-onboard — and to prioritize the draft nudge emails.

**What we need.** A last-active timestamp (and ideally a coarse activity level) per member, and the same for pending invites where available (invited-not-activated). Could be folded into an enriched `list_configuration` or `get_engagement_metrics` (#2).

**Acceptance criteria.**
- Each active member carries a last-active date (or "no activity recorded").
- Pending invites distinguish "invited, never signed up" from "signed up, never active."

---

## 4 — Pipeline & stage definitions / CRM schema (read) · P1

**Title:** Public MCP: read pipeline definitions, stage criteria, and CRM schema

**Problem.** A Coach agent is only as good as its fluency in *this company's* process. `/design-agent` and the Coach archetype need the pipeline definition(s), stage names and entry/exit criteria, and the custom-property schema to write process-aware prompts. Today the analyst infers process from opportunity data and asks the operator to confirm — workable, but lossy.

**What we need.** A read tool (parallel to the relevant slice of the internal admin `get_workspace_sales_context`) returning pipelines, their stages (with any defined criteria), and the CRM object/custom-property schema. Read-only.

**Acceptance criteria.**
- Lists each pipeline with ordered stages and any stage definitions/criteria.
- Lists custom properties per object type, with which are writable.
- Large payloads can be fetched in sections (avoid forcing a huge single blob into context).

---

## 5 — Object completeness / `updatedAt` in search results · P2

**Title:** Public MCP: expose `updatedAt` and field-completeness in object search

**Problem.** The CRM Data Nerd archetype is about keeping records correct and complete. To measure hygiene at scale (stale opportunities, opps missing key fields, contacts without notes), the analyst needs last-updated timestamps and a sense of field completeness on objects returned by `search_objects` — rather than reading every object individually.

**What we need.** Confirm whether `search_objects` already returns per-object `updatedAt` and the populated fields. If yes, document it and close this. If no, add `updatedAt` (and optionally a completeness/missing-required-fields summary) to search results.

**Acceptance criteria.**
- Search results carry `updatedAt` per object.
- A cheap way to identify stale or incomplete objects in a result set without N follow-up reads.

---

## How to file these

Each section above maps to one Linear issue — copy the **Title**, **Problem**, **What we need**, and **Acceptance criteria** into the issue body, set the priority from the table, and link them under a parent epic (suggested: *"Public MCP — agent-value analyst support"*). When one ships, update the analyst (`.claude/agents/data-analyst.md`, Dimension 3) and `/agent-audit` to use it, and check the box here.
