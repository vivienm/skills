---
name: gh-pr-merge-squash
description: >-
  Merge a GitHub PR by squashing locally, rebasing onto its up-to-date target,
  pushing with a lease, and using rebase merge with branch deletion.
license: MIT
---

# GitHub PR squash merge

Unless instructed otherwise, prepare one commit locally, rebase it onto the
latest PR target, push it with an explicit lease, then merge with
`gh pr merge --rebase --delete-branch`.

A request to merge authorizes this workflow, including rewriting the PR branch
and deleting it after merging. Do not ask again for these defaults. A request
only to inspect or prepare a PR does not authorize merging.
Explicit user instructions and repository rules take precedence.

## Identify the PR and protect local work

- Resolve the requested PR, its head and target repositories and branches,
  and its remote head SHA. Use the actual target, usually `main`; do not
  silently retarget the PR or assume a remote named `origin`.
- Stop for a draft or closed, unmerged PR. If already merged, report its result
  without rewriting or merging again.
- Inspect the diff, local status, branch tips, and worktrees. Work from a clean
  checkout of the verified PR head. Preserve uncommitted and ignored files and
  local-only commits; do not include unrelated work or discard it to proceed.
  Use protected switches such as `git checkout --no-overwrite-ignore`.
- If the PR is part of a stack or has dependent PRs, use `gh-stack` for the
  stack workflow instead. If that skill is unavailable, stop and report the
  missing dependency before rewriting or pushing. Do not squash multiple PRs
  together or delete a branch still needed by another PR. A non-default target
  alone is not a stack.
  Do not merge other PRs without explicit instructions; report any unmerged
  predecessors before proceeding.
- Before rewriting or pushing, verify that the repository allows rebase merging
  and that any required merge queue can honor this workflow. If either is
  incompatible, stop and report the limitation rather than changing methods.

## Prepare and push

1. Fetch the target and PR head from their verified remotes. Record the head
   SHA before rewriting; if it differs from the version inspected, review the
   new changes first.
2. Squash only the PR's commits into one local commit, then rebase that commit
   onto the fetched target. Choose a meaningful message from the final diff
   under repository commit conventions; use an empty body unless context is
   useful. Skip squashing if there is already one suitable commit. Resolve
   conflicts within the PR's scope and verify the resulting diff.
3. Run relevant repository checks on the final result, then push to the PR's
   head branch using the recorded remote SHA as the lease:

   ```sh
   git push --force-with-lease="refs/heads/$head_branch:$old_head_sha" \
     "$head_remote" "HEAD:refs/heads/$head_branch"
   ```

   If the lease fails, fetch and inspect the new remote work before proceeding;
   never retry with plain `--force` or merely replace the expected SHA.

## Validate and merge

Check CI, reviews, and mergeability on the pushed head, and recheck
compatibility with the repository's merge methods and any required merge queue.
Wait for pending required checks for up to five minutes unless instructed
otherwise. Report remaining blockers without bypassing rules, using `--admin`,
changing repository settings, or switching merge methods.

Immediately before merging, refresh the PR and confirm its head SHA and target
still match the prepared version. If either changed, reassess before proceeding.
Use the pushed SHA for the merge guard:

```sh
gh pr merge "$pr_url" --rebase --delete-branch \
  --match-head-commit "$head_sha"
```

Omit `--delete-branch` if deletion or the CLI's local checkout changes would
endanger local work, remove a branch with extra commits, or interfere with
another worktree; report the retained branches. Do not force cleanup.

After a failure or uncertain response, read the PR state before retrying.
Confirm GitHub reports the PR merged; queued or auto-merge-enabled is not yet
merged. Fetch the target and confirm the resulting commit is present. When safe,
fast-forward the local target with `git merge --ff-only --no-overwrite-ignore`;
leave a dirty or diverged checkout untouched.

Report the PR URL, resulting commit, and any incomplete local synchronization
or branch deletion. GitHub rebase merge can change the commit SHA; do not assume
the local prepared SHA is the resulting commit.
