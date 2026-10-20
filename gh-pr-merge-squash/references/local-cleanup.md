# Verify, synchronize, and clean up

Read the file-protection section before any worktree update. Follow the full
procedure after GitHub confirms the PR is merged, including when it was
already merged before this run. Complete verification before local
synchronization or deletion.

## Protect local files

A clean `git status` does not account for ignored files. Protect ignored
files during every worktree update, including branch switches and
fast-forwards. Use `git checkout --no-overwrite-ignore` (with `-b` or
`--detach` as needed) and `git merge --ff-only --no-overwrite-ignore`, or
equivalent protected operations. If ignored files would be overwritten,
preserve them and skip the affected synchronization or deletion; report
incomplete cleanup.

## Verify the merged result

Confirm that GitHub reports the PR as merged. Retrieve its resulting
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

## Refresh dependencies before deletion

Repeat the [stack and dependent discovery](../SKILL.md#detect-stacks-and-dependents)
before deleting the local topic branch, even if none were found before merging.
If discovery cannot be completed, retain the branch. If stack evidence or
dependencies were found before or after merging, apply the
[stack reference's reconciliation and retirement rules](stacked-prs.md#reconcile-dependents-and-retire-the-merged-branch),
including refreshing every recorded dependent PR.

## Synchronize the local target

Before switching to or updating an existing local target, resolve its upstream
remote's repository identity and verify the landing repository and branch.
A matching branch name or shared history is insufficient, especially with an
obsolete intermediate stack base. If the mapping is wrong or unverifiable,
leave the branch and upstream unchanged and report incomplete synchronization.
Otherwise:

- If checked out in another worktree, synchronize there only if clean;
  leave a dirty worktree untouched rather than checking out its branch here.
- Otherwise, use a clean current worktree and switch to the local target,
  creating it from the fetched remote-tracking ref if absent.

Fast-forward against that ref using the protected merge above. If the target
has extra commits, diverges, or cannot be updated safely, preserve it and report
incomplete synchronization; never reset or force a merge.

## Check the local topic branch

Before deleting the local topic branch, verify that it is distinct from
the local target, still points to the exact PR head actually merged,
and has no uncommitted work in a worktree using it. If that head cannot
be established or the local tip differs, retain the branch and report
why. Do not infer integration from a missing upstream or from commit
ancestry alone after a squash.

## Delete the local topic branch

If the topic branch is checked out in the current clean worktree, switch
to the local target before deletion only if its identity was verified.
If the target's identity could not be verified or it is already checked
out elsewhere, detach the current worktree at the verified fetched
target commit instead. If another worktree holds the topic branch,
retain it and report that fact. Never remove a worktree as incidental
cleanup. After these checks, delete the local topic branch. A squash may
require `git branch -D` because the original commits are not ancestors
of the target; use it only after proving integration and rechecking the
local branch tip. The dependency checks above must also pass.
