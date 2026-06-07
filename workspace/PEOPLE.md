# Who's Who in the Workspace

> **What this is:** the cast — everyone in the Day AI workspace, who they really are, and what they own. The agents read this to know who they're configuring agents and skills *for*. It's joined at the hip with the plan: every outcome owner appears here, and every key player has outcomes they own.
>
> **How it's built:** `/setup` and `/plan` populate this via the `gtm-strategist`, which reads `manage_workspace_members → list_configuration` (roster + roles), `assistant_settings → list` (who has which agent), and `list_suggested_invites` (who's in the CRM but not yet a member). Confirm and correct it in the interview — the agents trust it.
>
> **Keep it candid but careful.** This file is *not* synced to Day AI Pages by default (see `/sync-pages`) precisely so it can hold honest working notes. Don't put anything here you wouldn't want a teammate to read if it ever were shared.

**Last updated:** {date} · **Workspace:** {name} · **Domains:** {claimed domains}

---

## Members

> One entry per active workspace member. "Role" = their real daily job, not just a title. Mark inferences with *(inferred — confirm)*.

### {Full name} — {real role}
- **Email:** {email}
- **Workspace role:** {Owner / Admin / Member}
- **Agent:** {agent name, or "none — no seat yet"}
- **Owns:** {what they're responsible for — ties to OUTCOMES.md}
- **How to work with them:** {one line — comms style, what to route to them, what not to}

---

## Not yet in the workspace

> From `list_suggested_invites` and the plan. Candidates to bring in — the operator decides who and at what role.

| Name | Email | Why they belong | Proposed role |
|------|-------|-----------------|---------------|
| | | | |

## Pending & declined invites

| Name / email | Role | Status | Note |
|--------------|------|--------|------|
| | | pending / declined | |

## Coverage notes

- {gaps: an owner with no agent, a segment with no owner, a key person who's a Member but needs Admin, etc.}

---

<details>
<summary>Example entry (delete this)</summary>

### Jordan Pici — Account Executive (the first human a prospect meets)
- **Email:** jordan@acme.com
- **Workspace role:** Member
- **Agent:** "Ada" (Professional tier)
- **Owns:** Mid-market outbound pipeline; runs 5–8 demos/week
- **How to work with them:** Fast-moving relationship-builder. Route live-deal context and prep to them; don't route admin or data entry. Skills should be punchy and Slack-delivered.

</details>
