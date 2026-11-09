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

Read the target PR's head and base repositories and branch names. Match the
remotes to those repositories, accounting for forks and non-default bases.

Check the worktree for local changes and in-progress operations. Resume a paused
merge or rebase only if it belongs to the target PR, using the original tool in
the worktree holding it. Start new operations in a clean worktree with no
operation in progress. Use a separate worktree to preserve unrelated changes or
operations when needed.

Check stack membership using `gh stack view --json` or GitHub stack metadata.
Match the target by PR URL when available, or by head branch and repository. The
local command shows only the current stack, so a missing match is inconclusive.
A non-default base alone is insufficient evidence of a stack. Resolve uncertain
membership before choosing a workflow.

Preserve the intent of both changes. Check resolved paths for conflict markers,
stage only those paths, and confirm no unmerged paths remain before continuing.

## Stacked PR

Use the `gh-stack` skill for checkout, remote selection, worktree ownership, and
recovery. Report a missing skill or extension before attempting stack
operations. Use a cascading rebase to preserve the stack's linear ancestry.

Check out the lowest branch that needs rebasing and run
`gh stack rebase --upstack`. If it stops on conflicts, resolve and stage them in
the reported worktree without creating a separate commit. Continue with
`gh stack rebase --continue` instead of `git rebase --continue`. Repeat until
the rebase completes.

`--upstack` limits the operation to the current branch through the top. If
changes to the trunk require lower layers to be refreshed too, use
`gh stack rebase` for the whole stack instead. If `gh stack sync` reports a
conflict and restores the stack, recreate the conflict with `gh stack rebase`
before resolving it. Use `gh stack rebase --abort` when abandoning the
operation.

## Standalone PR

Check out the PR head and incorporate any remote head updates without discarding
local work. Set `base_remote` and `base_branch` to the PR's actual base remote
and branch, then fetch and merge that branch:

```shell
git fetch "$base_remote" "refs/heads/$base_branch" &&
  git merge --no-edit FETCH_HEAD
```

If the merge stops on conflicts, resolve and stage them, then finish the pending
merge with `git commit --no-edit`. Use `git merge --abort` to abandon the merge.

## Verify and publish

Inspect the result and run relevant checks. Before publishing, compare each
outgoing branch with its current remote head. Verify that all commits the push
would add or replace are in scope. `gh stack push` covers all active branches,
even after a rebase limited with `--upstack`.

When publishing is in scope, use `gh stack push` for a stack. For a standalone
PR, push normally to its head repository and branch using
`git push "$head_remote" "HEAD:refs/heads/$head_branch"`. After a failed push,
check which remote refs changed and reconcile before retrying. Resolving
conflicts does not authorize merging or closing the PR.

Report the workflow used, affected branches, checks, and publication status of
each branch, including any partial push.
