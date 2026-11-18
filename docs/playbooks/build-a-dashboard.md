# Playbook: build a live pipeline dashboard

> For when an outcome needs a number people can watch, such as pipeline by
> stage, created vs. won, what slipped, or campaign response. In Day AI that's
> a **Page of live widgets**, not a scheduled report and not an SDK app. The
> capability reference is in
> [`../PRODUCT_FACTS.md`](../PRODUCT_FACTS.md#live-dashboards-on-pages).

## Decide the shape first

- **Who reads it, and what do they decide from it?** A CRO's Monday view, a
  manager's team view, and a rep's own pipeline are three different pages.
  Write the 4–6 questions the page answers before calling a tool. A question
  you can't name a decision for is a widget you don't need.
- **Self-serve or built for them?**
  - **One person wants their own dashboard:** point them to Day AI's
    **Pipeline Dashboard** library template (`/pipeline-dashboard`,
    Professional or higher). It builds a private page of 5–6 live widgets
    and can rebuild it on demand. This is usually the right answer; don't
    rebuild it by hand.
  - **The team needs a shared dashboard bound to the plan** (initiative
    targets, the operator's stage definitions): build it from this harness,
    as below.
- **Never schedule a dashboard.** Widgets recompute on every page load. A
  skill that "refreshes the dashboard" wastes an automated slot. Schedule a
  skill only to *narrate* what changed, and have it link the page.

## Build it (connected, Owner/Admin, preview first)

1. **Ground the definitions.** `search_objects` on `native_pipeline` (sales
   pipelines only; leave out recruiting, partner, and suggested deals) and
   `native_stage` for stage ids and order. Run `read_crm_schema` on
   `native_opportunity` for the forecast or amount field and any custom
   fields. Copy ids verbatim; never type them.
2. **Draft each widget as a question.** The proven pipeline set:
   - **Open pipeline by stage:** `create_widget`, sum of value grouped by
     stage, open stages, pipeline order.
   - **Created vs. won by month:** `create_widget` with a `query` on
     `native_opportunity`, `asOf: {every: "month"}` for 6 months, grouped by
     `snapshotAt`, counting `createdInPeriod` and `wonInPeriod`.
   - **What moved in the last 30 days:** a `query` on
     `native_opportunity.changes`, grouped by movement (New, Advanced,
     Slipped, Close date pushed, Won, Lost…).
   - **Start-to-end walk for the quarter:** `native_opportunity.bridge`.
   - **When deals expect to close:** open deals by expected close date
     bucket (overdue / this month / next month / later).
   - **Where deals stall:** average `daysInStage` by open stage.
   - **Appendix:** `create_table_widget`, the 10 open deals longest in stage.
   Put the question in `chart.title` and let the tool pick the chart.
3. **Lay it out.** Lead with a `<day-ai-dashboardgrid data-columns="3">` of
   KPI tiles. Then use 2-column rows pairing related charts, full-width for
   stage charts, and the table last. Add a `<day-ai-panel variant="callout">`
   for one line of context ("Live: recomputes on open. Suggested deals
   excluded."). Never put numbers in the page text, because the widgets
   carry the data.
4. **Embed with the full recipe.** Over the MCP, use `recipe="…"` copied
   exactly from each widget tool's result; `recipe-ref` only resolves inside
   Day AI chat. Keep page HTML under ~8,000 characters, or use
   `pageHtmlFileId`.
5. **Share deliberately.** Pages are private by default. For a team
   dashboard, file it in a workspace-shared folder or set
   `publishedForUserAt`. Each viewer's widgets run with that viewer's own
   access, so a shared dashboard can show different people different totals
   if their visibility differs. Tell the operator.
6. **Record it.** Add the page title and id to the initiative or outcome it
   serves, so `/brain-health` can check the binding.

## Preview contents

Show the operator, for each widget, the question, the source, measure, and
grouping, and which field ids it uses. Also show the page layout and the
sharing. Get approval before `create_page`.

## Pair it with a skill (optional)

The flywheel version: the dashboard page is the shared truth, and a weekly
skill on the pipeline owner's agent attaches the page (`attachmentChanges`)
and writes the narrative: "three deals slipped, here's why." The skill reads
the page; the page never needs the skill.

## Guides

Day AI has no dedicated dashboard guide yet (check `../GUIDES.md`). For the pipeline-hygiene practice that makes a dashboard trustworthy, link [Keep Your Pipeline Accurate](https://day.ai/guides/keep-pipeline-accurate).

## When it isn't a Page

Reach for `/build-app` (the SDK) only when the view must live outside Day
AI, mash Day AI data up with a system Day AI can't see, or needs
interactions a Page can't host.
