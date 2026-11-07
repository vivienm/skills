---
name: github-conflicts
description: >-
  Resolve merge conflicts in GitHub pull requests. Use gh-stack's cascading
  rebase for stacked PRs and merge the PR's base branch into standalone PR
  branches.
license: MIT
---

# GitHub conflicts

## Identify the branches and workflow

Identify the target PR, its head branch and repository, and its actual base
branch and repository. Do not assume the base is `main` or `master`, or that
head and base use the same remote; fork PRs often use different remotes.

Inspect the working tree and any operation already in progress. Resume a
resolution only if it belongs to the target PR, using the original tool in the
worktree holding the paused operation. Leave unrelated operations and changes
intact; use a separate clean worktree when needed. Start new merges or rebases
from a clean worktree and stage only resolved paths.

Determine stack membership by matching the target PR's head branch and
repository against local gh-stack state (`gh stack view --json`) and GitHub
stack metadata as needed. The command reports the current stack, which may not
contain the target. A non-default base does not by itself imply a stack. Missing
local tracking or a failed lookup does not prove that the PR is standalone.
Resolve uncertain stack membership before choosing either workflow.

During resolution, preserve the intent of both changes rather than blindly
accepting either side. Check each resolved path for leftover conflict markers
before staging it, and confirm no unmerged paths remain before continuing.

## Stacked PR

Load and follow the `gh-stack` skill for stack checkout, remote selection,
worktree ownership, and recovery. If the skill or extension is unavailable,
report the missing dependency before attempting stack operations. Keep the
stack's linear ancestry by using a cascading rebase rather than merging the base
branch into a stack layer.

Check out the branch at the start of the range that needs rebasing:

```shell
gh stack rebase --upstack
# If the rebase stops on conflicts, resolve them in the reported worktree, then:
git add -- <resolved-paths>
gh stack rebase --continue
```

Repeat resolution and continuation until the operation completes. Do not create
a separate resolution commit or substitute `git rebase --continue` for the stack
continuation command.

`--upstack` limits the operation to the current branch through the top. If
changes to the trunk require lower layers to be refreshed too, use
`gh stack rebase` for the whole stack instead. If `gh stack sync` reports a
conflict and restores the stack, recreate the conflict with `gh stack rebase`
before resolving it. Use `gh stack rebase --abort` when abandoning the
operation.

## Standalone PR

Merge the latest base into the PR's head branch, following GitHub's
[Resolve conflicts workflow](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/resolving-a-merge-conflict-on-github).

Check out the PR head, incorporate any remote head updates without discarding
local work, then fetch and merge the actual base:

```shell
git fetch <base-remote> <base-branch> && git merge --no-edit FETCH_HEAD
# If the merge stops on conflicts, resolve them, then:
git add -- <resolved-paths>
git commit --no-edit
```

Finish the merge commit only when a merge is pending. Do not rebase or
force-push a standalone PR by default. Use `git merge --abort` when abandoning
the merge.

## Verify and publish

Inspect the resulting changes and run relevant repository checks. Before
publishing, compare each outgoing branch with its remote head and confirm that
all outgoing commits belong to the requested work; a clean worktree does not
guarantee this. `gh stack push` covers all active branches in the stack, even
after a rebase limited with `--upstack`.

When publishing is in scope, use `gh stack push` for a stack or a normal
`git push <head-remote> HEAD:<head-branch>` for a standalone PR. If a push
fails, inspect remote state before retrying; a stack push may already have
updated some branches. Reconcile remote changes rather than overwriting them.
Resolving conflicts does not itself authorize merging or closing the PR.

Report the workflow used, affected branches, checks, and publication status of
each branch, including any partial push.
