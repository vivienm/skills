---
name: gh-pr-merge-squash
description: Merge a requested GitHub pull request as one commit with linear history, then fast-forward the local base branch and safely delete the local topic branch.
license: MIT
---

# GitHub PR squash merge

Merge the requested PR as exactly one commit on its current destination,
without a merge commit; then fast-forward the local target and safely delete
the local topic branch. Identify the actual destination and its repository's
remote; do not assume `main` or `origin`. Explicit user instructions and
repository rules take precedence.

A merge request authorizes this workflow, subject to the confirmations below.
Do not ask otherwise or repeat confirmation for the same PR and intent.
A request to inspect or prepare a merge does not authorize merging.

## Establish the scope

- Resolve the repository and PR from the user's reference or current branch;
  resolve ambiguity before merging. Record the URL, base and head repositories
  and branches, exact head SHA, and commit count of this PR, not the whole
  stack or an unrelated local branch.
- Stop for a draft or closed, unmerged PR. For an already merged PR, use its
  recorded result and proceed to verification and cleanup: skip merge-only
  readiness, stack eligibility, and intent checks, but still discover
  dependents and apply all cleanup safeguards.
- Determine the base repository's default branch from current GitHub metadata,
  not the fork's default branch. Stop if it cannot be determined.
- Inspect the diff, current checks, current review decision, mergeability,
  and repository merge rules. Historical requests for changes are not
  necessarily outstanding objections.
- Inspect local branches, remotes, `git status`, and `git worktree list`.
  Identify the actual PR head branch; a matching name alone is insufficient,
  especially for forks. Preserve uncommitted work and local-only commits.

### Detect stacks and dependents

Read the PR's REST resource, including `stack`, on the verified host and base
repository; local `gh stack view --json` cannot rule out a remote stack.
The predecessor traversal stops at the native stack's declared base, or
otherwise the base repository's default branch unless user or repository
instructions establish another trunk.

With pagination, always find open dependent PRs in this PR's head repository,
including forks; verify their `base.repo` and `base.ref` match its head repository
and ref. When its base differs from the trunk, follow unmerged predecessor
candidates whose head repository/ref matches each successive base; include closed,
unmerged PRs. Stop traversal at the trunk even if PRs originate there. Check
local stack catalogs and identified local dependents; record dependent PRs'
base refs and head SHAs before merging.

If stacks, candidate predecessors, local dependencies, or uncertain stack
evidence appear, read [Stacked PRs](references/stacked-prs.md) before mutation.
Qualify historical candidates there before treating them as dependencies.
Otherwise do not load it. API errors are not evidence of absence; resolve
uncertain membership before merging when stack evidence exists.

## Confirm the merge intent

Before modifying branches, merging, or cleanup, ask if a merge remains to be
performed and either condition applies:

- Its actual destination differs from the base repository's default branch.
  Name both, explain the possible stack, and confirm the target, even when
  this skill was explicitly invoked. Do not retarget or merge dependencies
  instead. For native stacks, apply the reference's destination and eligibility
  rules before asking.
- This skill was selected implicitly, the PR has multiple commits, and the
  user has not explicitly requested or confirmed squashing this PR. State the
  count and ask to combine them. Explicit invocation of `$gh-pr-merge-squash`
  or a direct request to use this named skill expresses squash intent.

A generic "merge this PR" confirms neither condition. Combine both questions
when applicable and wait for the answer. Already merged PRs need neither
confirmation for the remaining synchronization and guarded cleanup.

## Merge on GitHub

Prefer GitHub's squash merge without first rewriting the topic branch. Do not
rebase just for linear history. If rules, conflicts, or requested validation
require updating the branch, use the appropriate workflow and recheck its
head; any necessary force-push must use a lease against the inspected head.
Before any worktree update, read and apply [local file protection](references/local-cleanup.md#protect-local-files),
including protection of ignored files during switches and fast-forwards.

Before submission, wait for applicable pending checks for up to five minutes
unless the user specifies another duration. Poll with backoff from five to at
most thirty seconds between requests. Start this window when first waiting
for checks in this run; status or head changes do not restart it. At the
deadline, refresh readiness once. If checks remain pending or readiness
cannot be established, report the PR URL and latest known check states,
then stop without submitting a merge or cleaning up branches.

Report failures, outstanding review requirements, conflicts, or unavailable
readiness information; do not bypass them or use administrator overrides.
Report blockers requiring changes beyond the merge request rather than
silently expanding scope.

Respect a configured merge queue: its method takes precedence over `--squash`.
Before auto-merge or enqueueing, verify it satisfies the single-commit
requirement; stop if incompatible or unverifiable.

Choose a meaningful subject from the final diff and PR, following repository
commit conventions. Add a body only for useful additional context. For direct
merges, supply the message explicitly, including an empty body when appropriate;
do not concatenate intermediate messages blindly. If an exact message is
required, establish that the merge path can honor it before submitting, or
report the limitation. Verify the actual resulting message during cleanup.

Immediately before merging, refresh the PR and verify the checked head SHA,
target repository and branch, and stack membership/order are unchanged. If
changed, refresh scope and readiness, re-evaluate squash confirmation, and
repeat target confirmation when required. For stacks, recheck eligibility
under the reference; stop if another unmerged PR would be included.

For an ordinary PR:

```sh
gh pr merge "$pr_url" --squash --match-head-commit "$head_sha" \
  --subject "$subject" --body ""
```

For a useful body, use `--body-file "$body_file"` with an agent-written body.
For native stacks, use the [asynchronous merge procedure](references/stacked-prs.md#submit-and-track-the-merge)
directly. Do not pass `--delete-branch`: remote deletion requires an explicit
request or the repository's automatic policy.

After a failed or uncertain attempt, read the PR state before retrying; do not
retry an unchanged blocker. If the head changed, inspect and check it again
and re-evaluate squash confirmation. Queued or auto-merge-enabled is not merged:
read [Wait for merge completion](references/merge-completion.md) for asynchronous,
queued, auto-merge, or resumed requests before observing completion or retrying.

## Verify, synchronize, and report

Once GitHub confirms the merge, read and follow [local verification and cleanup](references/local-cleanup.md),
including for already merged PRs. It covers the resulting commit and message,
fresh dependency discovery, protected worktree updates, fast-forwarding the
verified local target, and guarded local topic deletion. Apply the stack
reference's reconciliation and retirement rules when dependencies were found
before or after merging.

Report the PR URL, resulting commit, local target status, and whether the topic
branch was deleted or retained and why. Distinguish merge success from incomplete
cleanup; never hide errors. Report actual commit-message discrepancies and
dependent PR retargeting or remote head rewrites, including local branches
remaining on old history and needing stack synchronization.
