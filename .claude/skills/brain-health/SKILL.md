---
name: brain-health
description: The GTM Brain's own health loop. Builds a binding manifest of what the brain's files reference (people, pages, properties, skills, agents), compares the brain against the live workspace, and writes a proposals-only report of drift, breakage, and improvement opportunities. Never applies a change. Run weekly, or whenever the org or the workspace shifts. Usage: /brain-health [focus]
---

# /brain-health

The brain describes a business, and businesses move: people change roles, pipelines gain stages, pages get reorganized, skills reference owners who left. This skill is the loop that notices. It watches, compares, and **proposes; it never applies.** Every change it suggests goes through the operator and `/implement`'s preview gate like any other change.

$ARGUMENTS

If arguments name a focus (a person, an agent, "bindings", "delivery"), scope to that. Otherwise run the full pass.

---

## Step 1: Build the binding manifest

Scan the repo's markdown (planning docs, PEOPLE.md, TECH_STACK.md, PRIVACY.md, initiatives, preflight payload if present) and record every **binding**: a reference from a brain file to something live. Four kinds:

- **People bindings:** a named owner, DRI, SME producer, or activation owner.
- **Workspace-object bindings:** a page, folder, property, pipeline, or stage a skill or doc references by name or id.
- **Skill/agent bindings:** which skills the brain believes exist on which agents, with which triggers.
- **Capability bindings:** MCP tools the harness's own procedures depend on.

Write the manifest to `rollouts/health/manifest.md` as `file:line → binding → what depends on it`. The manifest is what makes the rest of the pass cheap and precise, and other skills may consult it ("does anything reference this page before I move it?").

## Step 2: Compare against the live workspace

Requires the MCP connected; in pre-signup mode, skip to the repo-only checks in Step 3.

- **Roster drift:** `list_configuration` vs `PEOPLE.md`. Departures, arrivals, role changes, owners named in the plan who aren't members.
- **Fleet drift:** `assistant_settings → list` and `manage_skills → list` vs what the brain believes. Skills deleted or disabled, identities edited away from their specs, agents on unexpected tiers.
- **Delivery health:** for scheduled skills, `manage_skills → get_history`: firing recently, substantive output, `notification.delivered` true (a `null` `notification` means no send was attempted: a configuration gap, not a failed delivery). A skill that's configured but not delivering is drift of the most expensive kind.
- **Object drift:** do the pages, folders, and properties the manifest binds to still exist where the brain thinks they are (`read_page`, `read_crm_schema`)?
- **Capability drift:** tools the harness depends on that have changed shape or disappeared from the connected tool list.

## Step 3: Apply the philosophy filter

**The default answer is "no change."** The brain is intentionally thin, and most observations don't warrant touching it. Propose an update only when one of three conditions holds:

1. **A binding broke or drifted:** a skill references a person who left, a page that moved, a property that was renamed; a spec and the live config disagree.
2. **A recorded gap closed:** something the brain noted as impossible (in an initiative's deferred list, or a prior report's can't-verify section) is now possible, and an initiative wanted it.
3. **A pattern changed:** the business's own structure shifted (new pipeline stage, methodology change, a new SME owning a guide) in a way the brain's documents should reflect.

Everything else is an observation, not a proposal. A new tool in the stack that nothing binds to is a line in the report, not a TECH_STACK.md edit.

## Step 4: Report, proposals only

Write to `rollouts/health/<YYYY-MM-DD>.md`:

```markdown
## Brain health: {date}

**Manifest:** {N} bindings across {N} files · **Checks run:** {list} · **Verdict:** {healthy | N proposals}

### Broken or drifted bindings
| Binding | Where (file:line) | What changed | Proposed fix |
|---------|-------------------|--------------|--------------|

### Delivery health
{Scheduled skills not firing / not delivering / producing hollow output, with evidence from get_history.}

### Proposals
{Each: what to change, in which file or via which /implement scope, and the evidence. Ranked.}

### Observations (no action)
{What moved that the brain correctly ignores.}
```

Present the report, and where a proposal is repo-only (fix a stale owner in PEOPLE.md), offer to make the edit with the operator's approval. Anything workspace-side routes to `/implement`.

---

## Notes

- **Never auto-apply.** Not even the trivial fixes. A background loop that edits the brain silently is how trust in the brain dies; the operator reviews, always.
- **Cadence:** weekly is right for most teams. This skill also makes a good scheduled skill on the operator's own Day AI agent once connected, delivering the report as a page or Slack post; pair it with a scheduled `/audit` for the plan-vs-workspace lens. The living-guide flywheel covers content freshness; this loop covers structural health. Three loops, three jobs.
- **Honest gaps are findings.** Checks you couldn't run (permissions, missing tools) get listed as unverified, not skipped silently.
