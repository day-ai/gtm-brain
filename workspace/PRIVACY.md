# Privacy & Sharing

> **What this is:** the privacy posture per persona, the exclusion rules, and the sign-off. Gate 4 of `map-your-gtm`.
>
> **The hard rule:** nothing in `rollouts/preflight/` deploys and no connector wires until this file carries a named sign-off with a date. Privacy comes before dataflows, always. It sits fourth in the discovery conversation only because tiers are decided per persona, so the roster (Gate 3) has to exist first.
>
> **What this file can and can't do:** workspace-level exclusions and rules can be applied at connect time. Per-user email and meeting sharing settings live with each user and **cannot be pushed via MCP**; for those, this file is the source of intent, and its deployable output is the per-user setup instructions in `rollouts/preflight/ENABLEMENT/`.

**Last updated:** {date} · **Sign-off:** {name, role, date (or TODO: unsigned)}

---

## Tiers by persona

> The same workspace serves the CEO's comp threads and a rep's deal emails. Decide, per persona, what their agent may read and what their teammates may see, before any account connects.

| Persona | Who (from PEOPLE.md) | Email sharing | Recording mode | Meeting sharing (internal / external × 1:1 / group) | Rationale |
|---------|----------------------|---------------|----------------|-----------------------------------------------------|-----------|
| Leaders | {names} | {workspace / private + rules} | {all / internal-only / external-only / none} | | {e.g. board + comp threads stay private} |
| Managers | {names} | | | | |
| Frontline | {names} | | | | |

## Workspace-level exclusions

> Domains and addresses whose mail never enters the customer memory, regardless of user settings.

| Exclusion | Type | Why |
|-----------|------|-----|
| {e.g. hr-vendor.com} | domain | HR matters |
| {e.g. counsel@lawfirm.com} | address | Legal privilege |
| {investor relations, personal domains, ...} | | |

## Guest and external access

- {who outside the claimed domains may see what, if anything}

## Sign-off record

> Who approved this posture. Legal/compliance if the business has them; otherwise the exec sponsor.

- {name, role} approved on {date}: {scope of approval}

## What each user does at rollout

Generated per-user instructions live in `rollouts/preflight/ENABLEMENT/`; each user applies their own sharing settings in the first session (their agent can open the right screen with `open_email_sharing_rules`). The activation owner in PEOPLE.md verifies completion; a skipped privacy walkthrough is the disengagement we know shows up weeks later.
