---
name: github-conflicts
description: >-
  Resolve GitHub pull request conflicts by merging the base into a standalone PR
  or rebasing a stack with gh-stack.
license: MIT
---

# GitHub conflicts

## Choose the workflow

Verify that the PR is open and identify its head and base repositories and refs.
Determine whether the PR belongs to a stack before choosing a workflow. Look for
the PR in `gh stack view --json` in the relevant checkout. If no local entry
matches, check the PR's `stack` field in the [GitHub REST API]. A failed lookup
leaves stack membership undetermined.

Start in a clean worktree and preserve unrelated work. Resume an operation only
if it matches the target PR and all affected branches are in scope. Use the
original tool. Resolve conflicts in line with the intent of both changes. Before
continuing, stage only the resolved paths and check for leftover markers and
unmerged paths.

## Standalone PR

Start from the freshly fetched PR head and include local commits only if they
are in scope. Fetch the PR's actual base and merge it into the head, as the
[GitHub conflict editor] does.

```shell
git fetch "$base_remote" "refs/heads/$base_branch" &&
  git merge --ff --no-edit FETCH_HEAD
```

Before resuming a merge in its worktree, verify the head and that `MERGE_HEAD`
contains only the expected base commit. Finish with `git commit --no-edit` after
staging the resolutions.

## Stacked PR

Follow the [gh-stack skill] for checkout, a local rebase scoped to the affected
layers, and conflict recovery. `gh stack sync` can publish.

## Verify and publish

Inspect the result and run relevant checks. Push only when authorized. Verify
that the outgoing commits and remote refs are in scope and that the updates
preserve remote work. For a standalone PR, push to its explicit head ref. For a
stack, `gh stack push` pushes all active branches, including those outside the
scope of a partial rebase.

After a failed push, inspect which remote refs changed before retrying. Report
the affected branches, checks, and publication status.

[gh-stack skill]: https://github.com/github/gh-stack/tree/main/skills/gh-stack
[GitHub REST API]:
  https://docs.github.com/en/pull-requests/reference/stacked-pull-requests-apis-and-webhooks#rest-api
[GitHub conflict editor]:
  https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/resolving-a-merge-conflict-on-github
