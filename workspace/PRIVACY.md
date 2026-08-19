# Privacy & Sharing

> **What this is:** how privacy works in Day AI, and the setup this team recommends per persona. Gate 4 of `map-your-gtm`.
>
> **How privacy actually works in Day AI:** every user controls their own sharing settings — email sharing (workspace vs. private, with rules by domain or address), recording mode, and meeting sharing — and applies them themselves when they activate. None of it can be pushed via MCP or enforced by this harness. So this file's job is **clarity, not enforcement**: explain the settings, record the setup the operator wants to recommend for each kind of user, and turn that into the per-user instructions in `rollouts/preflight/ENABLEMENT/` that each person follows in their first session. It sits fourth in the discovery conversation only because guidance is per-persona, so the roster (Gate 3) has to exist first.
>
> **Questions about Day AI's own security posture** (SOC 2, data handling, encryption): see **https://day.ai/trust**; the SOC 2 Type II report is available under NDA via the request form there. Record anything the trust center doesn't answer as an open question for your Day AI contact.

**Last updated:** {date} · **Guidance owner:** {name, role — whoever decided the recommended setup}

---

## Recommended setup by persona

> The same workspace serves the CEO's comp threads and a rep's deal emails, so one-size-fits-all guidance fails. If the operator wants to advise a setup across users, record it here per persona. Each user still applies — and can later adjust — their own settings.

| Persona | Who (from PEOPLE.md) | Email sharing | Recording mode | Meeting sharing (internal / external × 1:1 / group) | Rationale |
|---------|----------------------|---------------|----------------|-----------------------------------------------------|-----------|
| Leaders | {names} | {workspace / private + rules} | {all / internal-only / external-only / none} | | {e.g. board + comp threads stay private} |
| Managers | {names} | | | | |
| Frontline | {names} | | | | |

## Workspace-level exclusions

> Domains and addresses whose mail should stay out of the customer memory (HR, legal, investor relations, personal). Recorded as guidance: exclusions land as rules in each user's own sharing settings (see ENABLEMENT); anything that needs a workspace-wide mechanism is an open question for your Day AI contact.

| Exclusion | Type | Why |
|-----------|------|-----|
| {e.g. hr-vendor.com} | domain | HR matters |
| {e.g. counsel@lawfirm.com} | address | Legal privilege |
| {investor relations, personal domains, ...} | | |

## Guest and external access

- {who outside the claimed domains may see what, if anything}

## Decision record

> Who decided the recommended setup — legal/compliance if the business has them; otherwise the exec sponsor.

- {name, role} decided on {date}: {scope of the guidance}

## What each user does at rollout

Generated per-user instructions live in `rollouts/preflight/ENABLEMENT/`; each user applies their own sharing settings in the first session (their agent can open the right screen with `open_email_sharing_rules`). The activation owner in PEOPLE.md verifies completion; a skipped privacy walkthrough is the disengagement we know shows up weeks later.
