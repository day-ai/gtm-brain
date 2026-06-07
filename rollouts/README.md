# Rollouts

This is the audit trail. Every `/audit` and `/implement` run writes a dated folder here so you can see exactly what the workspace looked like, what changed, and how to undo it.

## Layout

```
rollouts/
├── 2026-06-07-audit/
│   └── REPORT.md                 # the gap report: plan vs. workspace
└── 2026-06-08-ae-briefings/
    ├── PROPOSED.md               # the full change set the implementor drafted
    ├── APPROVED.md               # what the operator actually approved
    ├── DEPLOYED.md               # results of each MCP call + timestamps
    └── snapshots/
        ├── <agent>-before.json   # assistant_settings read / manage_skills list, pre-change
        └── <agent>-after.json    # same, post-change
```

`/audit` writes `<date>-audit/REPORT.md`. `/implement` writes `<date>-<slug>/` with the proposed, approved, and deployed sets plus before/after snapshots.

## Why snapshots matter

Day AI keeps no separate version history for an agent — the `assistant_settings` `read` and `manage_skills` `list`/`get` output **is** the only record of what an agent looked like at a point in time. Capturing it before and after every change is how you diff a rollout and how you'd restore an agent if a change didn't land well.

## Restoring

To roll back, read the relevant `*-before.json` snapshot and re-apply those values with `assistant_settings update` / `manage_skills update`. The implementor can do this — point it at the snapshot and ask it to restore.

---

*Folders here are generated. They're safe to read, archive, or delete once a rollout is settled — but keep the most recent audit around, since `/implement` reads it.*
