---
name: setup-gtm-brain
description: >-
  Create a dedicated private GTM Brain repository with the complete harness,
  then hand off to /start. Use this before the first GTM Brain session; it
  never changes the current folder or the Day AI workspace.
---

# /setup-gtm-brain

Set up a new private GTM Brain workspace for the operator. This is the
**repository bootstrap**, not the business-work bootstrap: create the safe
private home for GTM Brain, then hand off to `/start` inside that repository.

Do not treat an arbitrary existing folder of plans as a GTM Brain repository.
The current harness owns root-level `CLAUDE.md`, `.mcp.json`, `.claude/`,
`initiatives/`, `planning/`, and `workspace/`; merging into existing files can
create a broken half-harness. This setup always creates a new directory.

$ARGUMENTS

Optional arguments may name the destination directory and private repository,
for example: `/setup-gtm-brain ./acme-gtm acme/gtm-brain`.

## What success looks like

The result is a newly created directory containing the complete GTM Brain
harness, with:

- `origin` pointing to the operator's new **private** GitHub repository;
- `upstream` pointing to `https://github.com/day-ai/gtm-brain.git`;
- the existing `CLAUDE.md`, `.mcp.json`, `.claude/` agents and skills,
  planning templates, initiatives, workspace file, docs, and rollout space;
- a clean Git worktree that shares history with `upstream`, so future harness
  updates can use `git pull upstream main`.

This setup does **not** authenticate Day AI, invite collaborators, inspect or
change the Day AI workspace, copy documents from the current folder, or run
`/start`.

## Step 1 — Gather the target and preflight

If the operator did not provide a destination, propose a new `./gtm-brain`
directory relative to the current working directory. If they did not provide
the private GitHub repository, ask for its `owner/repo` name. Do not assume the
authenticated GitHub user is the right owner; teams commonly need an
organization-owned private repository.

Before creating or changing anything, verify:

1. `git` and `gh` are available.
2. `gh auth status` succeeds.
3. The target directory does **not** already exist. If it does and contains
   both `.mcp.json` and `.claude/skills/start/SKILL.md`, say GTM Brain is
   already set up there and make no changes. Otherwise stop rather than
   merging into it.
4. `gh repo view {owner/repo}` does not already resolve. Do not attach to,
   overwrite, or change an existing GitHub repository.

Show the operator this plan and wait for explicit confirmation before the
GitHub write:

```text
I will clone the public GTM Brain harness into {target}, create the private
repository {owner/repo}, push the cloned history to it as origin, and retain
day-ai/gtm-brain as upstream. This will not access or change your Day AI
workspace. Proceed?
```

If any prerequisite is missing, stop with the smallest useful remedy. For
example, ask the operator to authenticate GitHub CLI with `gh auth login`.

## Step 2 — Create the private GTM Brain repository

After explicit confirmation, work in a fresh temporary directory beside the
target. Quote all paths and repository names. Do not use `gh repo create
--template`: the source repository is not an enabled GitHub template, and a
template-created repository would not share Git history for normal upstream
updates.

Use this shape of commands, substituting the confirmed values:

```sh
git clone --origin upstream https://github.com/day-ai/gtm-brain.git "{temporary-directory}"
gh repo create "{owner/repo}" --private --source "{temporary-directory}" --remote origin --push
mv "{temporary-directory}" "{target}"
```

If clone, repository creation, push, or move fails, do not delete a local
directory or a potentially-created GitHub repository. Explain what completed,
name the temporary path or remote, and give the operator the exact next command
to resume or clean up.

## Step 3 — Verify the handoff

Verify all of the following before declaring success:

1. `{target}/.mcp.json` exists.
2. `{target}/.claude/skills/start/SKILL.md` exists.
3. `git -C "{target}" remote get-url origin` is the new private repository.
4. `git -C "{target}" remote get-url upstream` is the public GTM Brain source.
5. `gh repo view "{owner/repo}" --json isPrivate` confirms `isPrivate: true`.
6. `git -C "{target}" status --short` is empty.

Then give only the next steps:

```text
Your private GTM Brain repository is ready.

1. cd {target}
2. Open it in Claude Code.
3. Approve the day-ai MCP server and complete OAuth.
4. Finish any Day AI chat onboarding, then run /start.
```

`/start` verifies the Day AI connection and the operator's role, takes stock of
the fresh `bootstrap-day-ai` initiative, and starts the actual GTM planning
work. It is intentionally separate from this setup skill.
