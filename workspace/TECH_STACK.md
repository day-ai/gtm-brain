# Tech Stack & Customer Memory

> **What this is:** every system a customer touches in this business, who owns each, what data lives where, and the migration posture. Gate 1 of `map-your-gtm`. `/discover` builds it from your inbox drops and interview; when a workspace connects, it becomes the connector plan.
>
> A criterion-met version of this file has no `TODO:` rows, a named owner per system, and a signed-off migration posture.

**Last updated:** {date} · **Migration posture:** {CRM Migration | Customer Memory | TODO: undecided} · **Sign-off:** {who, date}

---

## The migration posture

> **CRM Migration:** Day AI becomes the system of record; the incumbent CRM is exported and retired (typical arc ~8 weeks).
> **Customer Memory:** Day AI runs as the agent layer and memory system alongside the incumbent (typical arc ~6 weeks).

{Which one, and why. Who decided, and when.}

## Systems

> One row per system a customer touches. "Disposition" = stays / syncs / replaced.

| System | Category | Owner | Data that lives there | Disposition | Connector / integration notes |
|--------|----------|-------|----------------------|-------------|-------------------------------|
| {e.g. HubSpot} | CRM of record | {who} | {deals, contacts, fields} | {replaced} | {export path, API access, blockers} |
| {e.g. Gmail + Google Calendar} | Email / calendar | {who} | | stays | {per-user connection at rollout} |
| {e.g. Slack} | Team comms | {who} | | stays | {delivery layer for skills} |
| {e.g. Gong / Fathom / none} | Meeting recorder | {who} | | | |
| {warehouse / analytics} | | {who} | | | |
| {billing} | | {who} | | | |

## Data readiness

> Per source a planned skill will read: is the data **present**, **authoritative**, and **owned** (someone will vouch for it)? A skill planned against a source that fails any of the three is a failure mode, not a plan.

| Source | Present? | Authoritative? | Owned by | Notes |
|--------|----------|----------------|----------|-------|
| | | | | |

## Integration blockers

> Surface these now so they run in parallel with everything else, not as a mid-rollout surprise.

- {e.g. "Salesforce connector needs an API key; IT owns it; requested 2026-08-13"}

## Import plan pointers

Details live in `rollouts/preflight/IMPORT_PLAN.md`: what gets imported (open opportunities with amounts/owners/close dates, organizations, historical closed deals into a separate pipeline), from which exports, with which mappings.
