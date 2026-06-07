---
name: sync-pages
description: Sync the planning documents to/from Day AI Pages so the rest of the company sees the same source of truth. Pushes local planning/*.md to Pages, or pulls Pages back into the repo. Degrades gracefully if Pages tools aren't available on the connected MCP. Usage: /sync-pages [push|pull|status]
---

# /sync-pages

Keep the planning layer visible to the whole company by mirroring it to Day AI Pages. The repo is where the plan is *authored* and version-controlled; Pages is where the rest of the team *reads* it. This skill keeps the two in sync.

$ARGUMENTS

- **`push`** (default) — write the local `planning/*.md` documents up to Day AI Pages.
- **`pull`** — bring the Pages versions back into the repo (for edits made in Day AI).
- **`status`** — report what's linked and whether local and Pages versions differ, without changing anything.

---

## Step 0 — Check capability

Pages tooling depends on what the connected `day-ai` MCP exposes. Before doing anything, look at the available Day AI MCP tools for page operations (create/update/read/search a Page or document object).

- **If Pages tools are available:** proceed with the requested mode.
- **If they are not:** stop and tell the operator plainly: "The connected Day AI MCP doesn't expose Page tools in this session, so I can't sync automatically. Your plan lives in `planning/*.md` and stays the source of truth — you can paste these into a Day AI Page manually, or re-run `/sync-pages` once Page tools are available." Don't fail loudly or pretend it worked.

Never invent a Pages tool name. Discover it from the connected tool list.

---

## What gets synced

| Local file | Day AI Page |
|------------|-------------|
| `planning/COMPANY_PLAN.md` | "Company Plan" |
| `planning/STRATEGY.md` | "GTM Strategy" |
| `planning/OUTCOMES.md` | "GTM Outcomes" |

`workspace/PEOPLE.md` is **not** synced by default — it can contain candid notes about teammates that don't belong on a shared Page. Sync it only if the operator explicitly asks.

Track the link between each file and its Page (the Page ID and last-synced state) in `planning/.pages-sync.json` so future runs can detect drift. Create it on first successful push.

---

## push

1. For each planning document, read the local file.
2. If a linked Page exists (from `.pages-sync.json`), update it; otherwise create the Page and record its ID.
3. Preserve the document's markdown structure as faithfully as the Page format allows.
4. Report what was created vs. updated, with the Page links.

Before overwriting a Page that has changed since the last sync, **check for drift** — if the Page was edited in Day AI after the last push, warn the operator and ask whether to overwrite (push wins) or pull first. Don't silently clobber edits made in the product.

## pull

1. For each linked Page, read its current content.
2. Show the operator a diff against the local file.
3. On approval, write the Page content back into `planning/*.md`. Don't overwrite local edits without showing the diff first.

## status

Report, per document: linked Page (yes/no + link), whether local and Page differ, and which side changed since the last sync. Make no changes.

---

## Notes

- **The repo is the source of truth for authoring; Pages is for distribution.** When in doubt about which side wins, ask — don't guess.
- Pages is shared with the company, so the same messaging discipline applies: nothing on a synced Page should expose internal-only notes. That's exactly why `PEOPLE.md` is excluded by default.
- This is a convenience layer. If Pages tooling isn't available, the harness still works end to end — `/sync-pages` just becomes a manual copy-paste, and it says so.
