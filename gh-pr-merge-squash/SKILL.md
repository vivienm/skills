---
name: gh-pr-merge-squash
description: Merge a requested GitHub pull request as one commit with linear history, then fast-forward the local base branch and safely delete the local topic branch.
license: MIT
---

# GitHub PR squash merge

When asked to merge a PR, integrate its changes as exactly one commit on top
of its current target branch, without a merge commit. Then update the local
target branch by fast-forward and delete the local topic branch if safe.
Use the PR's actual destination and identify the remote for its base
repository; do not assume `main` or `origin`. For a native GitHub stack,
the landing destination is the stack base, which may differ from the PR's
immediate base branch.
Explicit user instructions and repository rules take precedence.

A request to merge authorizes this workflow, subject to the confirmation
rules below. Do not ask for confirmation when neither rule applies, or
repeat confirmation already obtained for the same PR and intent. A request
to inspect or prepare a merge does not authorize the merge itself.

## Establish the scope

- Identify the repository and PR from the user's reference or the current
  branch. Resolve ambiguity before merging. Record the PR URL, base and head
  repositories and branches, exact head commit SHA, and PR commit count.
  Count the commits in the actual PR, not the entire stack or an unrelated
  local branch.
- Check the PR state. Stop and report a draft or a closed, unmerged PR as
  not ready. If it is already merged, do not merge it again; use the recorded
  merge result and limit work to applicable synchronization and cleanup.
  Skip merge-only readiness, stack eligibility, and intent checks in that
  case; still discover dependents and apply all cleanup safeguards.
- Determine the base repository's default branch from current GitHub
  metadata; do not assume it is named `main` or infer it from the fork's
  default branch. If it cannot be determined, stop and report the issue.
- Inspect the PR diff, current checks, current review decision, mergeability,
  and repository merge rules. Use current review state rather than treating
  every historical request for changes as an outstanding objection.
- Inspect local branches, their remotes, `git status`, and `git worktree list`.
  Identify the local branch that actually corresponds to the PR head; a
  matching branch name alone is insufficient, especially for fork PRs.
  Preserve uncommitted work and local commits that are absent from the PR.
- Read the PR's GitHub REST resource, including its `stack` field; local
  `gh stack view --json` alone cannot rule out a remote stack. Also look for
  open PRs targeting this head branch, even without a native stack. Record
  dependent PRs' base refs and head SHAs before merging. For stack discovery,
  merge scope, and cleanup, read [Stacked PRs](references/stacked-prs.md).
  For a pending merge, only the lowest unmerged PR in a stack is eligible:
  verify that every lower PR is already merged and that the merge operation
  would include only the requested PR. Otherwise stop before mutation and
  identify the blocker. Do not offer confirmation for a combined merge,
  merge predecessors automatically, or retarget/unstack the PR to bypass
  this rule. Do not run a stack-wide synchronization as incidental cleanup.

## Confirm the merge intent

Ask for confirmation before modifying branches, merging, or performing
cleanup if either condition applies:

- A merge is still to be performed and its actual destination differs from
  the base repository's default branch.
  Name both branches, explain that this may be a stacked PR, and ask the
  user to confirm using that target for this PR's merge.
  For an eligible native stack PR, explain the immediate base and actual
  stack destination. This confirmation cannot make an upper PR eligible
  while a lower PR remains unmerged.
  This applies even when the skill was explicitly invoked. Do not
  automatically retarget the PR or merge its dependencies instead.
- A merge is still to be performed, the skill was selected implicitly, the
  PR contains more than one commit, and the user has not explicitly requested
  or confirmed squashing this PR. State the commit count and ask whether to
  combine those commits into one.
  Explicit invocation of `$gh-pr-merge-squash` or a direct request to use this
  named skill expresses squash intent; automatic skill selection does not.

A generic "merge this PR" confirms neither a non-default target nor the
squashing of multiple commits. When both conditions apply, ask for both
confirmations in one question. Wait for the required answer before proceeding.
For an already merged PR, these confirmations are not required for the
remaining synchronization and cleanup; apply the safety checks below.

## Merge on GitHub

Prefer GitHub's squash merge. It produces the desired single commit on the
current base without first rewriting the topic branch. Do not rebase merely
to obtain linear history. If repository rules, conflicts, or requested
validation require an updated branch, update it using the appropriate
workflow and recheck the resulting head before merging. Any necessary
force-push must use a lease against the inspected remote head.

Wait for applicable pending checks. Report failing checks, outstanding
review requirements, conflicts, or unavailable readiness information instead
of bypassing them. Do not use administrator overrides. If resolving a blocker
requires changes beyond the merge request, report it rather than expanding
the task silently.

If a merge queue is configured for the destination, respect it: its merge
method takes precedence over `--squash`. Before enabling auto-merge or adding the PR
to the queue, verify that this method satisfies the single-commit requirement.
If it does not, or cannot be verified, stop and report the blocker.

Choose a meaningful squash commit subject from the final diff and PR,
following repository commit conventions. Add a body only when it provides
useful context beyond the subject; omit it when the subject is sufficient.
For a direct merge, supply the message explicitly, including an empty body
when none is needed; do not blindly concatenate intermediate commit messages.
An empty body in the request is not proof that GitHub produced an empty body. If the user
requires an exact message, establish that the chosen merge path can honor
it before submitting; otherwise report that limitation.

Immediately before merging, refresh the PR and verify that the head commit
is still the one inspected and checked, and that the target repository,
branch, and stack membership/order have not changed. If any of these change,
refresh scope and readiness and repeat the confirmation checks above for
the new target when required. Recheck bottom-of-stack eligibility; stop if
another unmerged PR would now be included.
For an ordinary PR, with GitHub CLI:

```sh
gh pr merge "$pr_url" --squash --match-head-commit "$head_sha" \
  --subject "$subject" --body ""
```

When a body is useful, replace `--body ""` with `--body-file "$body_file"`
containing the agent-written body.

For a native GitHub stack, use the asynchronous merge API in the linked
reference directly. Do not first try `gh pr merge`, which does not support
that merge path. Avoid `gh stack merge` when its installed version cannot
bind the operation to the checked head SHA and explicit commit message, or
when its target could resolve to a stack number instead of the intended PR.

Do not pass `--delete-branch`: it also deletes the remote branch. Leave remote
deletion to an explicit user request or the repository's automatic policy.

If the head changes, inspect and check the new head and re-evaluate whether
squash confirmation is required before retrying. If a merge attempt fails
or its outcome is uncertain, read the PR state before retrying. Do not
repeatedly retry an unchanged blocker.
Queued or auto-merge-enabled is not yet merged: wait for completion or report
the pending state, and defer cleanup.

## Synchronize and clean up

A clean `git status` does not account for ignored files. Protect ignored
files during every worktree update, including branch switches and
fast-forwards. Use `git checkout --no-overwrite-ignore` (with `-b` or
`--detach` as needed) and `git merge --ff-only --no-overwrite-ignore`, or
equivalent protected operations. If ignored files would be overwritten,
preserve them and skip the affected synchronization or deletion; report
incomplete cleanup.

1. Confirm that GitHub reports the PR as merged. Retrieve its resulting
   commit SHA and the PR head that was actually merged, including after a
   queue delay or an uncertain merge attempt. Fetch the target from its
   verified remote and confirm that the resulting commit is reachable from
   the fetched target. Inspect the resulting commit's parents and full
   message (`git show -s --format='%P%n%B' "$merge_sha"`): verify one parent,
   the requested subject when applicable, and the actual body. For a PR
   already merged before this run, report the observed message without
   inventing a requested one. Where the pre-merge base is known, verify
   that exactly one commit represents this PR's integration;
   do not count unrelated commits that landed concurrently. Report a message
   mismatch as a completed merge with a discrepancy, distinguishing an empty
   body requested from one actually obtained. Never amend or force-push the
   published target to repair its message as incidental cleanup.
   Rediscover and refresh dependent PRs as described in the stack reference
   before deciding whether their old local parent branch can be deleted.
2. Before switching to or updating an existing local target, verify that its
   upstream corresponds to the verified landing repository and branch,
   rather than an obsolete intermediate base in a stack. Resolve the
   upstream remote's repository identity; a matching branch name or shared
   history is insufficient. If the upstream points elsewhere or the mapping
   cannot be established, leave the branch and its upstream unchanged,
   report incomplete synchronization, and skip its synchronization below.
   If the verified local target is checked out in another worktree,
   synchronize it there only if that worktree is clean. If it is dirty,
   leave it untouched and report incomplete synchronization; do not try to check out
   the target in the current worktree. If the target is not checked out in
   another worktree, use a clean current worktree and switch to the local
   target, creating it from the fetched remote-tracking ref if absent.
   In either safe case, update it with
   `git merge --ff-only --no-overwrite-ignore` against that
   ref. If the target has extra commits, diverges, or cannot be updated
   safely, preserve it and report incomplete synchronization; do not reset
   or merge it forcibly.
3. Before deleting the local topic branch, verify that it is distinct from
   the local target, still points to the exact PR head actually merged,
   and has no uncommitted work in a worktree using it. If that head cannot
   be established or the local tip differs, retain the branch and report
   why. Do not infer integration from a missing upstream or from commit
   ancestry alone after a squash.
4. If the topic branch is checked out in the current clean worktree, switch
   to the local target before deletion only if its identity was verified.
   If the target's identity could not be verified or it is already checked
   out elsewhere, detach the current worktree at the verified fetched
   target commit instead. If another worktree holds the topic branch,
   retain it and report that fact. Never remove a worktree as incidental
   cleanup. After these checks, delete the local topic branch. A squash may
   require `git branch -D` because the original commits are not ancestors
   of the target; use it only after proving integration and rechecking the
   local branch tip. For a stack, also satisfy the retirement criteria in
   the linked reference; the generic tip check alone is insufficient.

Never hide a synchronization or deletion error. Report the PR URL, resulting
commit, local target status, and whether the local topic branch was deleted
or retained with its reason. Distinguish a successful merge from incomplete
local cleanup. Report actual commit-message discrepancies and any dependent
PR retargeting or remote head rewrite, including whether the corresponding
local branch remains on its old history and needs stack synchronization.
