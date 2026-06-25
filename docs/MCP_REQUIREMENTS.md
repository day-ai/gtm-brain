# MCP Requirements — gaps the GTM Brain needs closed

The `data-analyst` agent and `/agent-audit` can already evaluate a lot from the existing Day AI MCP (membership, agent coverage, skill prompt craft, agent identity) and — with `get_skill_history`, now specified in DAY-2585 — whether each skill is actually *firing, producing substantive output, and being delivered.* A few high-value judgments still can't be made from what's exposed: above all, **engagement depth** (does a human act on the delivered output?) and **activation** (who's a live user vs. a cold seat). The `agent-implementor` also has one provisioning gap: it can recommend template-specific new agents, but the public MCP cannot attach a template slug to an agent authorization yet.

This doc records the remaining gaps as paste-ready Linear tickets. The analyst stays explicit about what it can confirm (build quality + delivery) vs. what it can't yet (engagement depth, activation), and cites this file.

> **Scope note.** DAY-2585 items are treated as **live** in this harness — including `assistant_settings` `list` + `targetAssistantId`, `manage_skills` cross-agent, `list_suggested_invites`, `resend_invite`, and now **`get_skill_history`** (run output + delivery result). The items below are **net-new** beyond DAY-2585.

**Last updated:** 2026-06-25

---

## Priority order

| # | Capability | Unblocks | Priority |
|---|------------|----------|----------|
| ~~1~~ | ~~Skill run history (read)~~ | ~~is a skill *delivering*, not just configured~~ | ✅ **Specified in DAY-2585** — `get_skill_history`; treated as live |
| 2 | Engagement metrics (workspace + per-agent/user) | Dimension 3 engagement depth + Dimension 1 activation | **P0** |
| 3 | Member activity / last-active | Dimension 1 — dormant vs active, who to nudge | P1 |
| 4 | Pipeline & stage definitions / CRM schema (read) | Dimension 2 — process-fluent Coach agents | P1 |
| 5 | Object completeness / `updatedAt` in search results | Dimension 3 — measuring CRM hygiene at scale | P2 |
| 6 | Template-specific agent authorization | `/implement` can deploy the exact starting template it previewed | P1 |

---

## 1 — Skill run history (read) · ✅ Specified in DAY-2585

**Status:** No longer a gap — translated into the DAY-2585 spec as the **`get_skill_history`** tool (run output, firing times, and `notification.result` delivery), and treated as live in this harness. The analyst (`.claude/agents/data-analyst.md`, Dimension 3a.3) and `/agent-audit` now read it to confirm a skill is firing, producing substantive vs. hollow output, and being delivered — rather than caveating every effectiveness claim.

The original requirement is preserved here for traceability:

> **What we needed.** A read tool (parallel to the internal admin `get_skill_history`) that, given a skill (and `targetAssistantId` for Admin/Owner), returns the last N runs: timestamp, whether it fired on schedule, the run's output/transcript (or a substantive summary), and the **delivery result** of any notification call (delivered via Slack DM / email / failed) — not just the channel config.
>
> **Acceptance criteria.** Returns ≥5 recent runs with timestamps and output; surfaces the notification *result* (delivered/failed + channel) so delivery is confirmed from the event, not `slackNotificationChannels`; Admin/Owner can read any agent via `targetAssistantId`, a user their own; "never run" returns empty, not an error.

What it does **not** cover (still open below): whether a human actually *engages* with the delivered output — that's **#2, engagement metrics**, now the top remaining priority.

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

## 6 — Template-specific agent authorization · P1

**Title:** Public MCP: allow agent template selection when provisioning an agent authorization

**Problem.** `/agent-audit` and `/design-agent` can now recommend a starting template for a new agent, but `/implement` cannot apply that recommendation through MCP. `manage_workspace_members` can invite members and `navigate_to_billing` can send the operator to the UI, but no public MCP input accepts `agentTemplateSlug`. That means a template-specific recommendation must be deployed manually in the Day AI admin UI, or the MCP invite is generic/custom.

**What we need.** Extend the public provisioning path to accept an optional `agentTemplateSlug` when creating an assistant authorization for an existing member or invited email. The accepted standard slugs are Turbo (`sales-assistant`, `meeting-notetaker`, `user-researcher`), Professional (`bdr`, `account-executive`, `sales-operator`, `crm-data-entry-specialist`, `sales-coach`, `marketing-director`), and Executive (`lead-analyst`, `gtm-strategist`, `senior-product-manager`). Gated Super Agent slugs (`revenue-operations-manager`, `demand-generation-manager`) should only be accepted when the Super Agent SKU gate is enabled. `null`/omitted should mean custom.

**Acceptance criteria.**
- Admin/Owner can create an agent authorization for an existing member or email invite with `{ tier, agentTemplateSlug? }`.
- The selected slug is returned by the read/list shape until the user activates the authorization.
- Activation seeds the agent identity from the slug; invalid/missing slugs fall back to template selection or custom for the authorized tier.
- Changing the tier on an unactivated authorization clears the stored slug.

---

## How to file these

Each section above maps to one Linear issue — copy the **Title**, **Problem**, **What we need**, and **Acceptance criteria** into the issue body, set the priority from the table, and link them under a parent epic (suggested: *"Public MCP — GTM Brain support"*). When one ships, update the relevant GTM Brain agent/skill docs and check the box here.
