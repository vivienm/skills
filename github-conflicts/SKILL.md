---
name: github-conflicts
description: >-
  Resolve GitHub pull request conflicts by merging the base into a standalone PR
  or rebasing a stack with gh-stack.
license: MIT
---

# GitHub conflicts

## Identify the PR and workflow

Verify that the target PR is open. Identify its head and base branches and
repositories, including forks and non-default bases.

Check for local changes and in-progress operations. Start new operations in a
clean worktree and preserve unrelated work. Resume an operation only when it
matches the target PR, using the original tool in the worktree where it paused.

Check stack membership with `gh stack view --json` for the relevant checkout and
GitHub stack metadata as needed. Match by PR identity or by head branch and
repository. The local command describes the current stack, so an absent match is
inconclusive on its own. Resolve ambiguous membership before choosing a
workflow.

Resolve conflicts in line with the intent of both changes. Check for leftover
conflict markers, stage only resolved paths, and confirm no unmerged paths
remain before continuing.

## Standalone PR

Merge the PR's actual base into its head, as the [GitHub conflict editor] does.
Check out the PR head and fast-forward it to the published head if it is behind.
If the histories diverged, reconcile the incoming and local work before
proceeding. Fetch from the remote that matches the PR's base repository.

```shell
git fetch "$base_remote" "refs/heads/$base_branch" &&
  git merge --no-edit FETCH_HEAD
```

If the merge stops on conflicts, resolve and stage them, then finish the pending
merge with `git commit --no-edit`. Before resuming an existing merge, verify the
checked-out branch and that its sole `MERGE_HEAD` matches the expected base
commit. Use `git merge --abort` to abandon the merge.

## Stacked PR

Use the `gh-stack` skill for stack checkout, remotes, worktree ownership, and
recovery. Report a missing skill or extension as a blocker. Use
`gh stack rebase --upstack` from the lowest affected layer, or `gh stack rebase`
when refreshing the whole stack. Use rebase for local resolution because
`gh stack sync` can publish.

If the rebase pauses, resolve and stage the conflicts in the reported worktree.
Continue with `gh stack rebase --continue` and repeat as needed. Use
`gh stack rebase --abort` to abandon the cascade.

## Verify and publish

Inspect the result and run relevant checks. Push only when authorized. Before
pushing, confirm the destination and check that the outgoing updates preserve
remote work and affect only branches and commits in scope. `gh stack push` can
update other layers after an `--upstack` rebase.

For a stack, follow the `gh-stack` skill. For a standalone PR, push to its
explicit head ref.

```shell
git push "$head_remote" "HEAD:refs/heads/$head_branch"
```

After a failed push, inspect which remote refs changed before retrying. Report
the affected branches, checks, and publication status.

[GitHub conflict editor]:
  https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/resolving-a-merge-conflict-on-github
