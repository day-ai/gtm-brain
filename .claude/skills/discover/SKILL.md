---
name: discover
description: >-
  The guided-discovery engine for map-your-gtm. Reads everything the operator
  already has (their GTM repo, existing Claude projects and skills, connected
  MCP sources, file drops in discovery/inbox/), then interviews to fill only
  the gaps. Surfaces first-win pilot candidates early, walks the five gates
  (tech stack, strategy and owners, people and permissions, privacy, fleet
  design), and writes the planning layer, workspace docs, and preflight
  payload. Works with no Day AI workspace; in a connected workspace it runs as
  a retro-mapping pass grounded in live data. Usage: /discover [a gate name,
  "first-win", or a focus]
metadata:
  internal: true
---

# /discover

Discovery is how the brain earns the right to configure anything. The posture is a paid consultant's: read everything available before asking a single question, ask one question at a time with a best guess attached, tell the operator plainly what you can't verify, and stop when decision-ready. The operator should feel interviewed by someone who did the homework.

$ARGUMENTS

If arguments name a gate (`tech-stack`, `strategy`, `people`, `privacy`, `fleet`), `first-win`, or a focus, scope the run to that. Otherwise resume wherever `initiatives/map-your-gtm.md` says the map stands.

---

## Step 0: Detect mode and take stock

- **Pre-signup** (the Day AI MCP is absent or unauthorized): everything grounds in the operator's materials and the interview. This is the normal mode for a prospect.
- **Connected**: this is a retro-mapping pass. Ground each gate in the live workspace FIRST (`manage_workspace_members → list_configuration` for the roster, `read_crm_schema` for properties, `assistant_settings → list` for the current fleet, existing sharing configuration for privacy) and interview only where the workspace can't answer. Output becomes diffs between mapped intent and live state, applied incrementally through `/implement`'s preview gate.

Then read the current state of the map: `initiatives/map-your-gtm.md` (and `first-win.md` if it exists), the planning docs, `workspace/TECH_STACK.md`, `workspace/PRIVACY.md`, `workspace/PEOPLE.md`. Never re-ask what's already answered; open by telling the operator where the map stands and what this session should close.

## Step 1: Read before you ask

Ingest sources in this priority order. Every fact you take from a source is one you don't spend interview time on. Cite where each came from.

1. **Their existing GTM repo.** The contents of `discovery/inbox/` (files or symlinks the operator drops there): plans, playbooks, process docs, board decks. If the operator runs their GTM from a repo already, ask them to point you at it or drop it in. This is the single highest-value source; push for it.
2. **Pre-existing Claude assets.** Anything they've already taught Claude, anywhere:
   - an existing Claude Project's instructions and knowledge files (ask them to export or paste into the inbox)
   - `CLAUDE.md` / `AGENTS.md` in repos they work in with Claude Code
   - existing `.claude/skills/` definitions and agent files
   Strategy found here is inherited, not re-interviewed. Existing skill definitions are recorded now and **adapted** into Day AI skill form at Gate 5, never rewritten from scratch: the person already invested in that prompt, and the investment carries over.
3. **Connected MCP servers the operator already has** (Notion, HubSpot, Salesforce, Slack, Google Drive, whatever their Claude Code session exposes). Use them as live discovery sources: pull the org chart from Notion, the pipeline shape from their CRM, the process docs from Drive. Read-only; you are mapping their world, not changing it.
4. **File drops**: CRM exports, org charts, spreadsheets in the inbox.

If `discovery/inbox/` doesn't exist, create it (with its README). If nothing is available, say so and proceed interview-only; that works, it's just slower.

## Step 2: The outcome interview, then the first win

The interview backbone, for everything: **Outcome → Workflow → Bottleneck → Agent → Data → Source.** Start from what the business is trying to achieve, find the workflow that achieves it, find where it breaks, decide what an agent could own, identify the data that requires, and confirm that data's source is present, authoritative, and owned.

Interview rules (same as the strategist's):
- One question at a time, each with your best guess attached, so the operator corrects rather than composes.
- Anchor to the layers in order: goals before strategy before outcomes.
- Stop when decision-ready. Unknowns get a visible `TODO:` with the specific outstanding question.

**The first-win moment comes immediately after the outcome interview**, before any deeper gate. Propose 1 to 3 candidate workflows that are net-new to this business: things their current stack cannot do at all, not incremental improvements to reports they already have. Rank candidates by three filters, and show the ranking:

1. **Provable in a short window** (weeks, not quarters).
2. **Not integration-gated** (doesn't wait on an API key from IT or a connector that doesn't exist). Judge this from `docs/CONNECTORS.md`, not from guesses — a candidate whose data imports natively (Gong, Granola, CSV) is not gated. A call-evidence candidate is gated until the call-capture posture (Gate 1) gives it a source: notetaker, Gong/Granola import, or both; if the posture is "none," it's off the table and you say so.
3. **The data it reads is ready** (present, authoritative, owned).

The operator picks one and scopes it: themselves, or up to ~5 users. Write `initiatives/first-win.md` from `initiatives/TEMPLATE.md` with numeric success criteria in the mini-MAP form: **proof point, baseline, target, judge, by-date.** "We want to love it" is not a criterion; convert it. Everything this workflow needs gets marked `PRIORITY: first-win` when the payload is built, and it deploys first at connect time.

## Step 3: Walk the gates

Each gate ends with an explicit confirmation ("Gate 3 is closed: here's what we decided") logged to map-your-gtm's Log. Gates can interleave across sessions; privacy sits after the roster only because its guidance is per-persona.

### Gate 1: Tech stack + customer memory → `workspace/TECH_STACK.md`
Every system a customer touches: CRM of record (or its absence), email/calendar, Slack, meeting recorder, warehouse/analytics, billing, everything else. Per system: owner, what data lives there, and whether it stays, syncs, or gets replaced. Decide the migration posture: **CRM Migration** (Day AI becomes the system of record) or **Customer Memory** (Day AI runs alongside the incumbent). Record data readiness per source: present, authoritative, owned. Check every system against `docs/CONNECTORS.md` and record its disposition from that catalog — what syncs natively, what bridges via CSV import + MCP connector, what has no path today. Surface integration blockers now ("Salesforce connector needs an API key from IT") so they run in parallel, not as a mid-rollout surprise.

**Ask the call-capture question explicitly; never leave it implied.** "Do you record calls today, and with what?" Then make the owner choose the posture: the **Day AI notetaker** records (per-person recording modes are decided at Gate 4), an existing recorder **imports** (Gong and Granola both import; a recorder with no import path in `docs/CONNECTORS.md` is an integration blocker to flag), **both** (decide right now which tool auto-joins which calls, so two notetakers never land in the same meeting), or **none**. Record the posture and its owner in `workspace/TECH_STACK.md`. This decision propagates: it gates which first-win candidates are viable (call-evidence workflows need a capture source), it feeds Gate 4's recording modes, and it lands in each user's ENABLEMENT setup steps. "No recorded calls" is a legitimate workspace, but it must be a decision someone made, with its cost named, not a default nobody chose.

### Gate 2: Strategy + levers + domain owners → planning docs + `workspace/PEOPLE.md`
Spawn the **gtm-strategist** for the three-layer plan (its pre-signup reconnaissance reads the inbox and Claude assets instead of the workspace graph). Capture the top 1 to 3 business priorities explicitly; they will live in the workspace instruction so every agent carries them in chat. Name the domain owners and SMEs: who owns product knowledge, enablement, pricing, the sales process, CS playbooks, and what each owns specifically. These people become the producers in the living-guide flywheels; nominate a guide owner per playbook now, so the flywheel has a producer on day one.

**Then propose the doc-to-Page conversion list.** For each strategy or playbook doc read in Step 1 (an inbox drop, a Notion page, a process doc), propose a disposition and get an explicit yes or no per doc: **becomes a workspace Page** (converted into `rollouts/preflight/PAGES/`, owned by its nominated SME, usually because agents and skills will work against it every day) or **stays in its source system** (agents reference it via a connector; nothing moves). The inbox privacy promise holds absolutely: nothing is ever promoted to a Page without the operator saying so, doc by doc. Converted pages get freshened, not copied blind: fix what the interview revealed is stale, strip anything that shouldn't be workspace-visible (candid notes, comp, named-account commentary), and note the source doc in a provenance comment at the top. A playbook whose owner was just nominated as a flywheel producer with no Page for the flywheel to turn on is an unfinished conversion; flag it.

**Corpora convert whole; approval stays per-doc.** When the inbox holds a linked set — a docs folder, a site export, a playbook with diagrams — propose the whole tree at once (the Day AI folder structure it would become), but collect the yes/no doc by doc, and show what each "no" severs: links into a declined doc become plain text, flagged. On approval, conversion does the full extraction into `rollouts/preflight/PAGES/` per its README's format contract: images pulled into `PAGES/_assets/` (rasterized to PNG where the source is SVG or Mermaid — the upload API takes no vector formats), intra-corpus links rewritten to relative staged paths, the freshen/strip pass applied, provenance and owner recorded in frontmatter. **Connected mode:** close the gate by offering to upload the approved corpus now — `/sync-pages push context` — rather than waiting for a preflight apply; the operator's context should be in the workspace the day they say yes. `/discover` itself still never writes to the workspace.

### Gate 3: People + permissions → `workspace/PEOPLE.md`
The full roster: name, email, real daily job, intended Day AI role (Owner/Admin/Member). Name the rollout lead and the exec sponsor. Name a **per-seat activation owner**: the person accountable for each user actually turning their agent on (this is the gate that attacks the invited-user stall). Confirm the operator of this repo is recorded with their trust note (from `/start` Step 1).

### Gate 4: Privacy → `workspace/PRIVACY.md`
This gate is educational, not contractual. Every user applies their own sharing settings when they activate (none of it can be pushed via MCP), so the harness's job here is clarity. First explain how the settings actually work: per-user email sharing (workspace vs private, with inclusion/exclusion rules by domain or address), recording mode (all / internal-only / external-only / none), and meeting sharing (internal/external by 1:1/group). Then ask the one question that matters: **"do you want to advise a recommended setup across your users?"** If yes, record it per persona (leaders, managers, frontline) with the rationale; if no, record explicitly that settings are each user's own call. Note exclusion intent (HR, legal, investor relations, personal domains) as guidance too. Open questions are normal here, not blockers, and there is no signature to collect — the gate's deployable output is the per-user setup instructions in `rollouts/preflight/ENABLEMENT/`, which each person's activation owner walks them through in their first session.

For questions about Day AI's own posture (SOC 2, data handling, security program): point the operator at **https://day.ai/trust**. The SOC 2 Type II report is available under NDA via the request form there. Anything the trust center doesn't answer (DPA, residency, SSO, model-training posture), record as a named open item for the operator's Day AI contact; never guess.

Gate 4 sits after Gate 3 because the guidance is per-persona and needs the roster.

### Gate 5: Agent requirements + fleet design → `rollouts/preflight/AGENTS/` + `SKILLS/`
Map archetypes to roles using `/design-agent` and its spec format: every seller's baseline Coach + CRM Data Nerd, plus what the outcomes demand. One job per agent; 2 to 4 agents for a real team to start; a value-vs-cost case for every agent beyond the default, with tier and slot budget per agent, priced per CLAUDE.md's pricing rules (current figures from https://day.ai/pricing pre-signup or workspace billing when connected; show the full seats + tiers + slots math; flag anything the source can't answer). Wire delegation explicitly where one agent hands work to another. Triage the existing skills and workflows found in Step 1 per the disposition menu below — porting is one outcome, not the goal. Draft custom properties (what do you know about a deal that your CRM doesn't track?), each with how it populates, null-case first for AI-managed ones. Sort every rule discovered along the way into its instruction layer per `docs/INSTRUCTION_ARCHITECTURE.md`.

**Triage the existing skills and workflows — porting is one outcome, not the goal.** For each one, first sketch what the Day AI version would honestly be (which agent, which trigger given the platform's four schedule shapes and seven events, which tools, what survives), *then* give it one of six dispositions, each with its reasoning stated so the operator can push back:

- **Port** — scheduled or event-shaped, per-person, push-mode work Day AI's toolset covers. Adapt against the `/write-skill` bar.
- **Split** — one big workflow becomes two or three focused skills, or a skill plus a page.
- **Becomes a page** — it's really shared knowledge, not a task. Convert via Gate 2's per-doc approval; skills reference it.
- **Stays a Claude skill** — interactive, orchestrating, multi-source analysis, or operator-only work that would lose what makes it good if squeezed into a single Day AI prompt. This repo is its home; say plainly why the Claude version is the better one, and name what Day AI should carry alongside it, if anything (often a page it reads or writes against).
- **Needs the SDK** — wants a UI, an external cron, or a mashup: route to `/build-app`.
- **Drop** — Day AI does it natively, or the payload already covers it. Say which.

A "stays a Claude skill" or "drop" verdict is a healthy outcome, not a failed mapping — a workflow forced into a shape that can't run it becomes a skill that fires and disappoints, which costs more trust than an honest "keep running this here." Record every disposition with its one-line reason in map-your-gtm's Log.

## Step 4: Enforce the failure-mode gates

Before "ready to connect" can be declared, verify all five, and name any that fail:

1. **Numeric success criteria** everywhere: baseline, target, judge, date. On first-win especially.
2. **Data readiness**: no planned skill reads a source that isn't present, authoritative, and owned.
3. **Per-seat activation owner** named for every user in PEOPLE.md.
4. **Scope defer list**: expansion ideas surfaced during discovery are recorded in map-your-gtm's Log as deferred, never silently absorbed into the initial build.
5. **A hard calendar date** for when success is measured.

## Step 5: Report

Close every session with where the map stands:

```markdown
## Discovery status: {date}

**Mode:** {pre-signup | connected retro-map}  ·  **Sources read:** {inbox files, Claude assets, MCPs}
**First win:** {workflow + scope + by-date, or "not yet scoped"}

| Gate | Status | What's decided | What's open |
|------|--------|----------------|-------------|
| 1 Tech stack | ... | ... | ... |
| 2 Strategy + owners | ... | ... | ... |
| 3 People + permissions | ... | ... | ... |
| 4 Privacy | ... | ... | ... |
| 5 Fleet design | ... | ... | ... |

### Do next
- {the single best next move: the question to answer, the doc to drop in the inbox, the gate to close}
```

---

## Notes

- **/discover reads and writes the repo; it never writes to a workspace.** Connected-mode retro-mapping produces diffs for `/implement`; deploys go through its preview gate like everything else.
- **Never fabricate.** A stage, property, owner, or system you haven't confirmed gets a `TODO:` or `ASSUMED:` marker, exactly like the planning docs. Honest gaps are findings.
- **Respect the operator's colleagues.** Discovery output lands in a repo their teammates may read. Candid, careful, no notes about people that you wouldn't show them.
- **Pace it.** Session one should reach the outcome interview and first-win candidates. The gates land over later sessions; `/start` tracks progress against map-your-gtm's criteria each time.
