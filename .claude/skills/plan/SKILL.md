---
name: plan
description: >-
  Build or refresh the planning layer — the company plan, the CRO-level
  strategy, the concrete outcomes, and the who's-who. Runs the gtm-strategist
  through an interview loop and grounds the plan in the workspace graph. Usage:
  /plan [layer-1|layer-2|layer-3|people | a free-form focus]
metadata:
  internal: true
---

# /plan

Build or deepen the planning layer. This is where the business's goals, revenue strategy, and concrete outcomes become living documents the rest of the harness executes against.

$ARGUMENTS

If an argument names a layer (`layer-1`/`company`, `layer-2`/`strategy`, `layer-3`/`outcomes`, `people`) or a focus area, scope the run to that. Otherwise do a full refresh across all three layers and the cast.

---

## Step 1 — Read what exists

Read the current state of the plan before changing it:
- `planning/COMPANY_PLAN.md`, `planning/STRATEGY.md`, `planning/OUTCOMES.md`
- `workspace/PEOPLE.md`

Note what's already settled and what's marked `TODO:` or `ASSUMED:` — those open items are this run's priority. **Preserve what's good; deepen what's thin.** Don't rewrite settled sections without reason.

---

## Step 2 — Ground in the workspace

Spawn the **`data-analyst`** subagent to refresh the facts the plan should rest on:

```
Pull a current workspace snapshot for planning: people & roles, agent/skill configuration coverage,
pipeline shape (and whether it's maintained), and activity/coverage gaps. Follow your output format.
Flag anything stale or permission-gated.
```

This keeps the plan honest — forecasts tied to real pipeline, outcomes tied to real coverage gaps, owners tied to real people.

---

## Step 3 — Run the strategist's interview loop

Spawn the **`gtm-strategist`** with the data-analyst's snapshot and the existing plan:

```
Refresh the planning layer{, scoped to {layer/focus} if specified}. Here is the current workspace
snapshot:

{paste data-analyst output}

And the current plan documents are in planning/ and workspace/PEOPLE.md. Resolve the open TODOs,
deepen thin sections, and keep the three layers coherent — every Layer-3 outcome must ladder up to a
Layer-2 strategy and a Layer-1 goal, name an owner from PEOPLE.md, and (where it implies a workspace
change) state what it becomes. Run a focused interview — one question at a time, each with your best
guess. Return the planning update summary.
```

Run the interview in the main thread so the operator answers in real time. One question at a time, each carrying your best guess so they can correct rather than compose.

---

## Step 4 — Write the documents

The strategist writes `planning/*.md` and `workspace/PEOPLE.md` directly, filling the existing templates. After it returns:

- Show the operator a concise diff of what changed across the four documents.
- List any TODOs that remain open and the specific question blocking each.
- Confirm coherence: every outcome has an owner who's in the cast, and every layer ladders up.

---

## Step 5 — Point to execution

Close with the bridge to implementation:

```markdown
## Plan refreshed

- **Layer 1 (Company Plan):** {one line on the state}
- **Layer 2 (Strategy):** {one line}
- **Layer 3 (Outcomes):** {N} outcomes, {N} owned, {N} that imply a workspace change
- **Open TODOs:** {N}

### Next
- `/audit`      → see which outcomes the workspace already delivers and where the gaps are
- `/sync-pages` → publish the updated plan to Day AI Pages
- `/start`      → take stock of initiatives; if a strategy shift here implies a bounded, owned effort, capture it as a new `initiatives/<slug>.md`
```

---

## Notes

- **Plan only — no workspace writes.** `/plan` reads and lists the workspace (via the data-analyst and strategist) but never invites, edits agents, or creates skills. That's `/implement`.
- Keep the plan decision-ready, not exhaustive. A tight plan an implementor can act on beats a sprawling one.
- If the workspace snapshot contradicts the plan (e.g. the forecast assumes a pipeline that isn't maintained), surface the contradiction in the interview — don't quietly write around it.
- **Plan vs. initiative.** `/plan` keeps the standing intent and the fine-grained outcomes current. When the plan implies a *bounded, owned effort* with a deadline and a verifiable definition of done ("get the SE team live on Day AI by Q3"), that's an **initiative**, not an outcome — note it so it can be captured in `initiatives/` and driven by `/start`. See `initiatives/README.md`.
