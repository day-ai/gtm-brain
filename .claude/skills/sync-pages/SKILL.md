---
name: sync-pages
description: >-
  Sync the planning documents and active initiatives to/from Day AI Pages so
  the rest of the company sees the same source of truth. Pushes local
  planning/*.md and initiatives/*.md to Pages, or pulls Pages back into the
  repo. The context scope uploads the approved knowledge corpus in
  rollouts/preflight/PAGES/ as real folders and pages, images and cross-links
  included. Degrades gracefully if Pages tools aren't available on the
  connected MCP. Usage: /sync-pages [push|pull|status] [context]
metadata:
  internal: true
---

# /sync-pages

Keep the planning layer visible to the whole company by mirroring it to Day AI Pages. The repo is where the plan is *authored* and version-controlled; Pages is where the rest of the team *reads* it. This skill keeps the two in sync.

$ARGUMENTS

- **`push`** (default) — write the local `planning/*.md` documents and active `initiatives/*.md` up to Day AI Pages.
- **`pull`** — bring the Pages versions back into the repo (for edits made in Day AI).
- **`status`** — report what's linked and whether local and Pages versions differ, without changing anything.
- **`push context`** — upload the knowledge corpus staged in `rollouts/preflight/PAGES/` (Gate 2's approved doc conversions) as real Day AI folders and pages, images and page-to-page links included. `status context` reports the corpus's link/drift state without changing anything.

---

## Step 0 — Check capability

Pages tooling depends on what the connected `day-ai` MCP exposes. Before doing anything, look at the available Day AI MCP tools for page operations — **create, update, read, and search/list** a Page or document object.

- **If Pages tools are available:** proceed with the requested mode. Note specifically whether a **search or list** capability exists — `push` needs it to find existing Pages before creating (see below). If you can create/update but cannot search/list Pages at all, say so: you'll have to rely on `.pages-sync.json` alone and can't guarantee against duplicates, so confirm with the operator before creating any Page that isn't already linked.
- **If they are not:** stop and tell the operator plainly: "The connected Day AI MCP doesn't expose Page tools in this session, so I can't sync automatically. Your plan lives in `planning/*.md` and stays the source of truth — you can paste these into a Day AI Page manually, or re-run `/sync-pages` once Page tools are available." Don't fail loudly or pretend it worked.

Never invent a Pages tool name. Discover it from the connected tool list.

---

## What gets synced

| Local file | Day AI Page |
|------------|-------------|
| `planning/COMPANY_PLAN.md` | "Company Plan" |
| `planning/STRATEGY.md` | "GTM Strategy" |
| `planning/OUTCOMES.md` | "GTM Outcomes" |
| `initiatives/<slug>.md` *(status `NEW` / `IN_PROGRESS` / `SUCCEEDED`)* | "Initiative: {title}" |

**Initiatives are a first-class thing to publish** — an owned effort with a deadline and a verifiable definition of done is exactly what the rest of the company wants visibility into. Sync the active ones (`NEW`, `IN_PROGRESS`) and recently `SUCCEEDED` ones by default; skip `PAUSED`/`CANCELLED` unless asked. Before pushing an initiative, apply the messaging discipline below — strip or summarize anything in its **Context**/**Cost**/**sources** that's internal-only (forecast detail, candid notes, raw meeting ids); publish the title, status, timeframe, DRI, success criteria, and the plain-language *why*. `initiatives/README.md` and `TEMPLATE.md` are scaffolding — never synced.

`workspace/PEOPLE.md` is **not** synced by default — it can contain candid notes about teammates that don't belong on a shared Page. Sync it only if the operator explicitly asks.

Track the link between each file and its Page (the Page ID and last-synced state) in `planning/.pages-sync.json` so future runs can detect drift. Create it on first successful push.

---

## push

1. For each synced file (the planning documents and each active initiative), read the local file and determine its **target Page title** (`Company Plan`, `GTM Strategy`, `GTM Outcomes`, or `Initiative: {title}`).
2. **Resolve the target to an existing Page before creating anything** (see "Resolving a target" below). Only create a new Page when no existing one resolves.
3. If it resolved to an existing Page, **update** it (after the drift check). If nothing resolved, **create** it and record its ID in `.pages-sync.json`.
4. Preserve the document's markdown structure as faithfully as the Page format allows.
5. Report what was **created** vs. **updated** vs. **adopted** (an unlinked Page matched by title and newly linked), with the Page links.

Before overwriting a Page that has changed since the last sync, **check for drift** — if the Page was edited in Day AI after the last push, warn the operator and ask whether to overwrite (push wins) or pull first. Don't silently clobber edits made in the product.

### Resolving a target (do this before every create)

Resolve each target to a Page in this order, and **stop at the first hit**:

1. **By recorded ID.** If `.pages-sync.json` has a Page ID for this file, read that Page to confirm it still exists. If it does, that's the target — even if its title has since changed (e.g. a renamed initiative): update the existing Page rather than spawning a new one. If the recorded ID no longer resolves (deleted in the product), drop the stale entry and fall through.
2. **By exact title.** Otherwise, search/list Pages for one whose title **exactly** matches the target title — full-string, case-sensitive, not a `contains`/fuzzy match (so `Initiative: Expand to EMEA` never matches `Initiative: Expand to EMEA — phase 2`).
   - **Exactly one match** → *adopt* it: record its ID in `.pages-sync.json` and treat it as the linked Page (then update it with the drift check). This is what stops a fresh clone, a missing/stale `.pages-sync.json`, another operator's earlier push, or a hand-made Page from producing a duplicate.
   - **More than one match** → ambiguous. Do **not** guess and do **not** create. List the matches with links and ask the operator which to adopt (and to clean up the extras).
3. **No match anywhere** → create the Page, then record its ID.

**Never create a Page whose exact title already exists.** The recorded ID is the primary key; the exact-title search is the dedupe guard for when the link file can't be trusted. If the search/list capability is unavailable (per Step 0), you cannot run step 2 — fall back to ID-only and confirm with the operator before creating any unlinked Page.

## push context

The planning docs above are a flat mirror; `rollouts/preflight/PAGES/` is a **corpus** — folders, images, page-to-page links — staged by `/discover` Gate 2 under per-doc approval. `push context` applies it by executing **the canonical apply algorithm in `rollouts/preflight/README.md` (PAGES section)**, exactly — the algorithm lives there once so this skill and `/implement`'s preflight pages step can't drift. In brief: probe (including `get_page_image_upload_url` + `attach_page_image`, which are still rolling out — degrade to recorded placeholders, never pretend an image landed) → preview the full tree and get approval → folders → create/update pages (resolution rules identical to push above; each new pageId recorded in `PAGES/.pages-sync.json` immediately) → per-page finalize (upload/attach/embed each image; rewrite each cross-link to its chip or anchor form) → `read_page` verification.

The corpus ledger is `PAGES/.pages-sync.json` — same schema and dedupe role as `planning/.pages-sync.json`, plus any pending image attaches a degraded run left behind. The drift check applies here too: a context page edited in Day AI after the last sync gets the overwrite-or-pull question, never a silent clobber.

## pull

1. For each linked Page, read its current content.
2. Show the operator a diff against the local file.
3. On approval, write the Page content back into `planning/*.md`. Don't overwrite local edits without showing the diff first.

## status

Run the same **resolution** as push (by recorded ID, then exact title) but make **no changes**. Report, per document:
- **Linked Page** — linked by ID / would be *adopted* by title (exists but not yet linked) / none found.
- **Drift** — whether local and Page differ, and which side changed since the last sync.
- **Duplicates** — flag any target title that matches more than one Page; these need cleanup before push can sync them safely.

---

## Notes

- **The repo is the source of truth for authoring; Pages is for distribution.** When in doubt about which side wins, ask — don't guess.
- **Never create a duplicate Page.** Always resolve a target to an existing Page (by recorded ID, then exact title) before creating. `.pages-sync.json` is the primary key, but it can't be trusted alone — it's missing on a fresh clone, can be deleted, and won't know about Pages other operators or hand-edits created. The exact-title search is the guard that makes push idempotent and safe to re-run.
- Pages is shared with the company, so the same messaging discipline applies: nothing on a synced Page should expose internal-only notes. That's exactly why `PEOPLE.md` is excluded by default.
- This is a convenience layer. If Pages tooling isn't available, the harness still works end to end — `/sync-pages` just becomes a manual copy-paste, and it says so.
