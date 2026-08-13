# The preflight payload

Everything a workspace needs at launch, authored as files before the workspace exists. `/discover` builds this directory during `map-your-gtm`; `/implement` (preflight mode) applies it the day you connect. Every format here is one the post-signup harness already consumes, so nothing gets rewritten at signup: the map IS the onboarding.

**The privacy gate applies to the whole directory:** nothing here deploys until `workspace/PRIVACY.md` carries a named, dated sign-off.

## Layout

```
rollouts/preflight/
  WORKSPACE_INSTRUCTION.md     the one workspace-wide instruction, drafted to the 3000-char cap
  CUSTOM_PROPERTIES.json       property definitions + backfill plan
  PAGES/                       company/GTM knowledge pages (one .html or .md per page)
  INVITES.md                   roster + roles + activation owners
  AGENTS/<person>-<archetype>/
    AGENT.md                   the /design-agent spec (identity, value-vs-cost, starter skills)
    CREATION_CARD.md           the manual UI step: name, title, photo, tier
  SKILLS/<slug>.md             one skill per file (see format below)
  IMPORT_PLAN.md               data cleanup + import mappings, per source
  ENABLEMENT/                  rollout assets for humans (see below)
```

## Formats

**WORKSPACE_INSTRUCTION.md:** the full proposed text, at or under 3000 characters (count it; the cap is hard). At apply time the implementor reads the live record first and merges; on a fresh workspace the merge is trivial, but the discipline is the same because `update` replaces the entire text.

**CUSTOM_PROPERTIES.json:** an array of definitions matching `create_or_update_custom_property`: `name`, `description`, `objectTypeId`, `propertyTypeId`, `options` (for enums), `aiManaged`, `useWeb`. AI-managed properties carry their populating prompt in `description`, written null-case first (what the agent should conclude when the evidence isn't there). A sibling `backfills` list names which properties get `backfill_custom_property` runs after import.

**PAGES/:** each file is one page. First line is the title; body is the content. Pages that anchor a living-guide flywheel name their SME producer in a comment at the top. Include the visuals: the agent hierarchy and the key workflows with their owners, as SVG or Mermaid, so the team can see the design, not just read it.

**INVITES.md:** a table: name, email, role (Owner/Admin/Member), why they belong (plan reference), activation owner. This is the input to `invite_member`; every row also appears in `workspace/PEOPLE.md`.

**AGENTS/:** one folder per agent. `AGENT.md` is the standard `/design-agent` spec. `CREATION_CARD.md` exists because **agents cannot be created via MCP**: a human creates each agent in the Day AI UI (a seat and billing step), then the harness applies the identity and skills. The card is everything that human needs on one screen: agent name, title, photo suggestion, tier, and whose agent it is.

**SKILLS/:** one skill per file, frontmatter + prompt:

```markdown
---
name: {skill name}
slash_command: {unique per agent}
target: {person's email or "workspace_library"}
agent: {which of their agents}
trigger: SCHEDULE | EVENT | none
trigger_value: {cron + timezone, or event types}
notification: slack | email
deployment_mode: MANAGED | TEMPLATE      # workspace_library only
priority: first-win | standard
---

{the full prompt, written to the /write-skill bar}
```

Where a prompt needs live-workspace grounding that doesn't exist yet (real pipeline stages, actual page ids, a teammate's meeting cadence), mark the spot inline:

```
GROUND-AFTER-CONNECT: {what to look up and where it goes}
```

At apply time, every marker is resolved against the live graph **before** the skill is enabled. A skill with unresolved markers never goes live; a skill that fires generic output on day one is worse than one that fires a day later grounded.

**IMPORT_PLAN.md:** per source: the export file, the cleanup steps, the mapping to Day AI objects (opportunities with amounts/owners/close dates, organizations, people, historical closed deals into their own pipeline), and who vouches for the data.

**ENABLEMENT/:** the change-management half of the payload, for humans not MCP:

- `rollout-guide-<persona>.md`: per-persona, what your agent does, your first week, how to engage with the guides/emails/call structures it produces, how to set up your connectors, how to adjust your own skills.
- `privacy-setup-<person>.md`: generated from `workspace/PRIVACY.md`, the exact sharing settings each user applies in their first session (these cannot be pushed via MCP; the activation owner verifies completion).
- `first-24-hours.md`: what to expect. Email sync takes hours before skills have anything to read; workspace context generates overnight. Set the expectation so the quiet start reads as normal, not broken.

## Apply order

`/implement` preflight mode applies in dependency order, previewing at each step:

1. Verify operator role (`list_configuration`)
2. Verify the privacy sign-off exists (hard stop if not)
3. Workspace instruction (merge, then write)
4. Custom properties
5. Pages and folders
6. Invites
7. **Manual step:** a human creates each agent from its creation card
8. Agent identities (`assistant_settings update`)
9. Skills (`priority: first-win` first, each with its GROUND-AFTER-CONNECT markers resolved before enabling)
10. Imports, then backfills

Slices marked `priority: first-win` deploy first within every step they appear in, so the pilot users see grounded output in their first session.
