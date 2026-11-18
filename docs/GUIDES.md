# Day AI Guides: where to point for use cases and best practices

> **What this is:** a map from the use cases this harness works on to Day
> AI's published guides at **https://day.ai/guides**. When an operator or a
> teammate needs the *why* and *how* of a use case (how to keep the pipeline
> accurate, how to organize a team of agents, how to run outbound), **link
> the matching guide; don't restate it.** The guides are maintained by Day
> AI and improve without this repo changing.
>
> **Division of labor:** guides teach use cases and best practice.
> [`PRODUCT_FACTS.md`](PRODUCT_FACTS.md) is the source for product limits
> (tiers, slots, gates, caps); where a guide and PRODUCT_FACTS disagree on a
> limit, PRODUCT_FACTS (re-verified against the live product) wins. The
> playbooks in [`playbooks/`](playbooks/) are how *this harness* executes.
>
> **The index is live.** New guides appear at https://day.ai/guides. If a use
> case isn't mapped below, check the index before saying no guide exists,
> and add the row here.

**Last verified:** 2026-11-18, fetched from https://day.ai/guides (62 guides).

---

## Use case → guide

All URLs are `https://day.ai/guides/<slug>`.

### Getting started and deciding how to run Day AI

| Use case | Guide | Point here when… |
|----------|-------|------------------|
| What Day AI is | [what-is-day-ai](https://day.ai/guides/what-is-day-ai) | someone new needs the four-part model in one read |
| Replace the CRM or run alongside it | [is-day-ai-a-crm](https://day.ai/guides/is-day-ai-a-crm) | Gate 1's migration posture is being decided |
| AI-native vs traditional CRM | [ai-native-crm](https://day.ai/guides/ai-native-crm) | a stakeholder asks what's actually different about the record |
| Run on top of Salesforce or HubSpot | [add-ai-on-top-of-your-crm](https://day.ai/guides/add-ai-on-top-of-your-crm) | the posture is Customer Memory (the incumbent stays) |
| Migrate off HubSpot | [migrate-from-hubspot-without-losing-deal-history](https://day.ai/guides/migrate-from-hubspot-without-losing-deal-history) | the posture is CRM Migration from HubSpot; pairs with `IMPORT_PLAN.md` |
| Sizing for a real sales team | [is-day-ai-good-for-a-50-person-sales-team](https://day.ai/guides/is-day-ai-good-for-a-50-person-sales-team) | a team-scale rollout is being scoped |
| Per-agent pricing | [per-agent-vs-per-seat-pricing](https://day.ai/guides/per-agent-vs-per-seat-pricing) | the operator is comparing Day AI's bill to a per-seat tool (current prices still come from day.ai/pricing) |
| What connects | [what-does-day-ai-integrate-with](https://day.ai/guides/what-does-day-ai-integrate-with) | Gate 1, alongside `docs/CONNECTORS.md`: what to connect first |
| Security review | [ai-crm-security-review](https://day.ai/guides/ai-crm-security-review) | security or legal is running a review (with https://day.ai/trust) |

### Agents and skills

| Use case | Guide | Point here when… |
|----------|-------|------------------|
| Agent vs skill | [ai-agent-vs-skill](https://day.ai/guides/ai-agent-vs-skill) | someone is confusing the two in fleet design |
| What GTM agents are | [gtm-ai-agents](https://day.ai/guides/gtm-ai-agents) | introducing the agents-as-job-functions idea |
| Organize a team of agents | [organize-a-team-of-gtm-ai-agents](https://day.ai/guides/organize-a-team-of-gtm-ai-agents) | Gate 5 fleet design, delegation, the Chief of Staff pattern |
| Deploy one skill to every rep | [deploy-a-claude-skill-to-your-team](https://day.ai/guides/deploy-a-claude-skill-to-your-team) | a team-wide Managed or Template library skill; pairs with `playbooks/deploy-library-skills.md` |
| Skills that improve from use | [self-improving-gtm-workflows](https://day.ai/guides/self-improving-gtm-workflows) | explaining the living-guide flywheel or a producer skill |
| Evaluate agents before rollout | [grade-your-ai-agent-setup](https://day.ai/guides/grade-your-ai-agent-setup) | before `/implement` widens a pilot; complements `/agent-audit` |
| GTM engineer or agents | [hire-a-gtm-engineer-or-use-tools](https://day.ai/guides/hire-a-gtm-engineer-or-use-tools) | a leader is weighing headcount against agents |
| RevOps jobs as agents | [best-ai-tools-for-revops](https://day.ai/guides/best-ai-tools-for-revops) | designing agents for a RevOps lead (pipeline upkeep, hygiene, forecasts) |
| Agency practice | [build-a-revops-agency-practice-on-ai-agents](https://day.ai/guides/build-a-revops-agency-practice-on-ai-agents) | the operator is a RevOps agency running client workspaces |

### Everyday use cases (what a skill or agent delivers)

| Use case | Guide | Point here when… |
|----------|-------|------------------|
| Morning briefing | [ai-morning-briefing-deals-meetings](https://day.ai/guides/ai-morning-briefing-deals-meetings) | a daily briefing skill is the first win |
| Meeting prep | [ai-meeting-prep](https://day.ai/guides/ai-meeting-prep) | designing a Coach's pre-meeting prep |
| Pipeline accuracy | [keep-pipeline-accurate](https://day.ai/guides/keep-pipeline-accurate) | designing a CRM Data Nerd; stage entrance criteria |
| CRM updates from calls | [update-your-crm-from-call-recordings](https://day.ai/guides/update-your-crm-from-call-recordings) | the post-meeting record-update skill |
| Why the CRM is stale | [crm-is-low-resolution](https://day.ai/guides/crm-is-low-resolution) | making the case for a Data Nerd to a skeptical leader |
| Email into the record | [gmail-to-crm-without-logging](https://day.ai/guides/gmail-to-crm-without-logging) | reps ask whether they must BCC or log anything |
| Calls recorded elsewhere | [add-a-call-recorded-outside-your-crm](https://day.ai/guides/add-a-call-recorded-outside-your-crm) | call-capture posture includes recordings from another tool |
| Choosing a call recorder | [best-call-recorder-that-syncs-to-crm](https://day.ai/guides/best-call-recorder-that-syncs-to-crm) | Gate 1's call-capture decision |
| Why deals stall or are lost | [find-out-why-deals-stall-or-get-lost](https://day.ai/guides/find-out-why-deals-stall-or-get-lost) | a Coach or Pipeline Analyst's loss-review skill |
| Feature requests to the backlog | [feature-requests-from-sales-calls](https://day.ai/guides/feature-requests-from-sales-calls) | a product-feedback outcome (often with Linear) |
| Outbound sequences | [ai-crm-with-outbound-email-sequences](https://day.ai/guides/ai-crm-with-outbound-email-sequences) | a campaign is being planned; pairs with `playbooks/set-up-a-campaign.md` |

### Data, access, and governance

| Use case | Guide | Point here when… |
|----------|-------|------------------|
| What agents can read and write | [control-ai-agents-crm-access](https://day.ai/guides/control-ai-agents-crm-access) | someone asks how agent access is governed |
| Team email, safely | [give-ai-agents-access-to-team-email-safely](https://day.ai/guides/give-ai-agents-access-to-team-email-safely) | Gate 4 privacy, alongside `workspace/PRIVACY.md` |
| Permissions on company knowledge | [permission-company-knowledge-base-for-ai-agents](https://day.ai/guides/permission-company-knowledge-base-for-ai-agents) | shared pages and folders vs. private content |
| CRM data for outside agents | [give-ai-agents-access-to-crm-data](https://day.ai/guides/give-ai-agents-access-to-crm-data) | an outcome needs Claude or another agent reading customer memory |
| Claude Code with customer memory | [give-claude-code-customer-memory](https://day.ai/guides/give-claude-code-customer-memory) | the operator wants their own Claude tools on Day AI's MCP (this repo is one example) |
| Context graph concepts | [what-is-a-context-graph](https://day.ai/guides/what-is-a-context-graph), [what-is-a-company-brain](https://day.ai/guides/what-is-a-company-brain) | explaining "customer memory" to a technical stakeholder |

### Evaluation and comparisons (operator context only)

Day AI also publishes comparisons and roundups. Examples are
`day-ai-vs-salesforce-agentforce`, `day-ai-vs-attio`, `day-ai-vs-glean`,
`day-ai-vs-granola`, `salesforce-alternative-for-scale-ups`, and the `best-*`
rankings. Point a **prospect operator** to these during evaluation, only
when they ask. **Never** put them in skill prompts, agent descriptions,
pages, or anything teammates read. Workspace-facing text makes no competitor
comparisons (`.claude/skills/write-skill/SKILL.md`).

## Not covered by a guide yet

As of the last check, these have no dedicated guide:

- **Live dashboards** (widgets, report queries)
- **Choosing an agent tier**
- **Super** (Lead Finder, Custom Smart Properties)

Use the playbooks and PRODUCT_FACTS for these, and re-check the index:
when a guide appears, add it here and link it from the playbook.

## How to link

- Link inline at the point of need, with one clause on why: *"How to set
  stage entrance criteria: see Keep Your Pipeline Accurate
  (day.ai/guides/keep-pipeline-accurate)."* Don't add a reading list.
- In customer-facing outputs (enablement guides, nudge emails, rollout
  plans), link at most one or two guides, the ones that answer that
  person's next question.
- In a skill prompt, a guide link is fine where the agent should hand a
  person further reading. The agent still works from what it knows and the
  pages it's attached to, not the guide.
- If a guide's advice conflicts with this team's recorded decisions (their
  stages, their privacy guidance), the team's decision wins. Note the
  difference instead of linking silently.
