---
name: setup-gtm-brain
description: >-
  Create a dedicated private GTM Brain repository with the complete Claude Code
  harness, then hand off to the start skill. Install this
  one-time setup skill globally; it never edits an existing invoking project
  or the Day AI workspace.
---

# setup-gtm-brain

Set up a new private GTM Brain workspace for the operator. This is the
**repository bootstrap**, not the business-work bootstrap: create the safe
private home for GTM Brain, then hand off to the `start` skill inside that
repository.

Invoke this skill as `/setup-gtm-brain` in Claude Code.

## Installation scope

This is a one-time global setup skill, not a skill that belongs in whichever
project happens to be open. Install it for the coding agent you are using:

```sh
npx skills add day-ai/gtm-brain --global
```

`--global` is intentional. Without it, the Skills CLI installs into the
current project by default. The Skills CLI installs the setup skill for the
selected or detected agent; it does not install GTM Brain's internal operating
skills globally.

The repository this skill creates is a Claude Code harness with `CLAUDE.md`,
`.claude/`, and `.mcp.json` for the Day AI MCP connection and write-safety
contract.

The **installer** therefore changes user-level agent configuration. Once
installed, `/setup-gtm-brain` does not edit or merge into the invoking project:
it creates a separate, confirmed target directory and does not access or
change the Day AI workspace. Its default target, `./gtm-brain`, creates a new
child directory beside the current project after confirmation; it does not
modify the project's existing files.

Do not treat an arbitrary existing folder of plans as a GTM Brain repository.
The harness owns `CLAUDE.md`, `.claude/`, `initiatives/`, `planning/`, and
`workspace/`; merging into existing files can create a broken half-harness.
This setup always creates a new directory.

The invoking request may name the destination directory and private repository.
For example, use `/setup-gtm-brain ./acme-gtm acme/gtm-brain`.

### Local validation source override

For development or local validation only, the invoking request may append
`--source <local-git-directory-or-git-url>`. This replaces the public source
for this one setup run, so a test can exercise an explicitly committed local
revision without cloning `day-ai/gtm-brain` from GitHub. The operator must
explicitly provide and confirm this override; never infer it. In normal founder
setup, omit it and use the public source.

## What success looks like

The result is a newly created directory containing the complete GTM Brain
harness, with:

- `origin` pointing to the operator's new **private** GitHub repository;
- `upstream` pointing to `https://github.com/day-ai/gtm-brain.git`, or to the
  explicitly confirmed `--source` only during local validation;
- the Claude Code harness (`CLAUDE.md`, `.mcp.json`, `.claude/`), plus the
  planning templates, initiatives, workspace file, docs, and rollout space;
- a clean Git worktree that shares history with `upstream`, so future harness
  updates can use `git pull upstream main`.

This setup does **not** authenticate Day AI, invite collaborators, inspect or
change the Day AI workspace, copy documents from the current folder, or run the
`start` skill.

## Step 1 — Gather the target and preflight

If the operator did not provide a destination, propose a new `./gtm-brain`
directory relative to the current working directory. If they did not provide
the private GitHub repository, ask for its `owner/repo` name. Do not assume the
authenticated GitHub user is the right owner; teams commonly need an
organization-owned private repository.

Set `{source}` to `https://github.com/day-ai/gtm-brain.git` unless the
operator explicitly provided `--source`. For a local-directory override,
resolve it to an absolute path, verify it is a cloneable Git repository with a
clean worktree **checked out on `main`**, and record `{source_head}` from
`git rev-parse HEAD` before proceeding. This preserves the normal shared-repo
contract: the created repository's default branch is `main`, and future
updates can use `git pull upstream main`. For a Git URL, resolve and record
the expected `main` commit/ref before proceeding. Repeat the exact source and
revision in the confirmation. Treat this as a local test mode, not a different
public installation path.

Before creating or changing anything, verify:

1. `git` and `gh` are available.
2. `gh auth status` succeeds.
3. The target directory does **not** already exist. If it does, do not merge
   into it. If it appears to be GTM Brain, verify the full handoff contract in
   Step 3 before saying it is already set up; otherwise stop without changes.
4. `gh repo view {owner/repo}` returns a confirmed not-found result. Treat an
   authentication, permission, network, or API failure as inconclusive and
   stop. Do not attach to,
   overwrite, or change an existing GitHub repository.

Show the operator this plan and wait for explicit confirmation before the
GitHub write:

```text
I will clone the GTM Brain harness from {source} into {target}, create the
private repository {owner/repo}, push the cloned history to it as origin, and
retain {source} as upstream. This will not access or change your Day AI
workspace. I verified source revision {source_head}. The result will work in
Claude Code. Proceed?
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
git clone --origin upstream "{source}" "{temporary-directory}"
git -C "{temporary-directory}" rev-parse HEAD # must equal {source_head}
gh repo create "{owner/repo}" --private --source "{temporary-directory}" --remote origin --push
mv "{temporary-directory}" "{target}"
```

If clone, repository creation, push, or move fails, do not delete a local
directory or a potentially-created GitHub repository. Explain what completed,
name the temporary path or remote, and give the operator the exact next command
to resume or clean up.

## Step 3 — Verify the handoff

Verify all of the following before declaring success:

1. The Claude Code harness exists: `{target}/.mcp.json` and
   `{target}/.claude/skills/start/SKILL.md`.
2. `git -C "{target}" remote get-url origin` is the new private repository.
3. `git -C "{target}" remote get-url upstream` is `{source}`.
4. `gh repo view "{owner/repo}" --json isPrivate` confirms `isPrivate: true`.
5. The remote default branch contains the same commit as local `HEAD`.
6. `git -C "{target}" status --short` is empty.

Then give only the next steps:

```text
Your private GTM Brain repository is ready.

1. cd {target} && claude
2. Approve the day-ai MCP server and complete OAuth.
3. Run /start.
```

The `start` skill verifies the Day AI connection and the operator's role, takes
stock of the fresh `bootstrap-day-ai` initiative, and starts the actual GTM
planning work. It is intentionally separate from this setup skill.
