# Playbook: choose an agent's tier

> Used by `/design-agent` (the spec's **Tier needed** line), `/discover`
> Gate 5 (fleet budget), and `/agent-audit` (upgrade and downgrade
> recommendations). The gates themselves live in
> [`../PRODUCT_FACTS.md`](../PRODUCT_FACTS.md); prices always come from
> https://day.ai/pricing or the workspace's billing, never from this file.

The tier is the agent's job description in billing form: it decides which
tools the agent has and how many skills it can run without being asked. Pick
the **lowest tier that does the job**, and make the case for anything above
it.

## Step 1: List what the agent must do, then read off the floor

For each starter skill and each job the agent owns, take the highest gate it
hits:

| If the agent must… | Floor |
|--------------------|-------|
| Run 1–2 automated skills, draft emails, research the web, keep tasks | Turbo |
| Keep opportunities and pipeline current, manage custom properties or views, build live dashboards, run more than 2 automated skills, or work a campaign someone else set up | Professional |
| Create and launch campaigns, find contacts at target accounts, run more than 5 automated skills, or run a skill every 4 hours | Executive |
| Prospect net-new accounts (Lead Finder), run multiplayer or self-prospecting campaigns, or keep AI-populated custom properties current | Super (annual only) |

Then count slots. Every enabled scheduled or event skill uses one, and
**every campaign the agent works uses two.** A Professional agent with four
scheduled skills can't also work a campaign.

## Step 2: Check the workspace-level requirements

- A workspace needs **at least one Professional-or-higher agent** before
  teammates join free. A fleet that's all Turbo doesn't work.
- Campaign *creation* is gated on the **user's highest-tier agent**, not the
  agent working the campaign. One Executive agent for the campaign owner can
  be enough; the teammates working their slices need Professional.
- AI-populated properties need a **Super agent that owns each property** (up
  to 20 per agent). One Super agent (often the RevOps or Data Nerd agent)
  can own the workspace's whole set of smart properties. Don't put every
  seller on Super for properties.

## Step 3: Make the case for anything above the floor

Write one line per agent in the spec's **Value vs. cost**: the job the tier
unlocks, the work product it produces, and the price difference from the
tier below (live price and billing cadence stated). Typical honest calls:

- **A CRM Data Nerd** usually needs Professional (pipeline and property
  writes; 3–5 automated skills).
- **A Coach** usually needs Professional. It needs Turbo only if it only
  preps meetings and drafts follow-ups and the team keeps the pipeline
  elsewhere.
- **A Pipeline / Forecast Analyst** needs Professional (dashboards, pipeline
  analytics). It needs Executive only if it must refresh every 4 hours.
- **A Campaign Operator** needs Executive for the person who creates
  campaigns.
- **A Prospector** needs Super: a sales-led, annual-only decision. Recommend
  it only when net-new pipeline from lookalike prospecting is a stated
  priority with an owner, and say it requires an annual agreement.

## Step 4: Before recommending a downgrade

Read the agent's skills (`manage_skills → list`) first. A downgrade
**switches off** automated skills beyond the new slot count, and any every-4-hours
schedule. Campaigns the agent works count two slots each, and below
Professional it can't work them at all. List what would turn off, in the
recommendation.

## Guides

When the operator is comparing Day AI's bill to a per-seat tool, link [Per-Agent vs Per-Seat CRM Pricing](https://day.ai/guides/per-agent-vs-per-seat-pricing). Current figures still come from https://day.ai/pricing. There's no tier-choice guide yet.

## Never

- Recommend the **Partner** tier. It's staff-only.
- Quote a price from memory. If the live page can't be reached, present
  every figure as an unverified estimate with the link.
- Promise negotiated, volume, or pilot terms. Volume discounts exist on
  annual billing; anything beyond the published table is "confirm with Day
  AI."
