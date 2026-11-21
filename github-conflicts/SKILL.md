---
name: github-conflicts
description: >-
  Resolve GitHub pull request conflicts by merging the base into a standalone PR
  or rebasing a stack with gh-stack.
license: MIT
---

# GitHub conflicts

Verify that the PR is open and identify its head and base repositories and
branches. Check its `stack` field in the [GitHub REST API] to choose the
workflow.

## Standalone PR

If the API confirms the PR has no stack, fetch its head and base. Start from the
fetched head and merge in the fetched base, as the [GitHub conflict editor]
does. Resume the PR's pending merge in its checkout after verifying that its
head and merge parent match the PR head and base commits. Preserve unrelated
local work.

Resolve the conflicts and stage the resolutions. Before finishing the merge,
review the index and confirm that all staged changes belong to the merge. Run
relevant checks. If authorized, push to the PR's head repository and branch.

## Stacked PR

Compare local tracking with the GitHub stack and confirm which branches will be
affected before rebasing. Follow the [gh-stack skill] for checkout, cascading
rebase, and conflict recovery. Run relevant checks and publish only authorized
updates that preserve remote work. `gh stack push` pushes every active branch,
so push branches explicitly when only a subset is authorized. For rewritten
branches, pin `--force-with-lease` to the remote tips included in their
pre-rebase history.

[gh-stack skill]: https://github.com/github/gh-stack/tree/main/skills/gh-stack
[GitHub REST API]:
  https://docs.github.com/en/pull-requests/reference/stacked-pull-requests-apis-and-webhooks#rest-api
[GitHub conflict editor]:
  https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/resolving-a-merge-conflict-on-github
