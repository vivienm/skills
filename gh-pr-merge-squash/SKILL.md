---
name: gh-pr-merge-squash
description: Merge a requested GitHub pull request as one commit with linear history, then fast-forward the local base branch and safely delete the local topic branch.
license: MIT
---

# GitHub PR squash merge

Merge only the requested PR as one commit on its current destination, without
a merge commit; then fast-forward the local destination and safely delete the
local topic branch.
Do not assume `main` or `origin`. Explicit user instructions and repository
rules take precedence.

A merge request authorizes this workflow, subject to the confirmations below.
Do not ask otherwise or repeat confirmation for the same PR and intent.
A request to inspect or prepare a merge does not authorize merging.

## Establish the scope

- Resolve the repository and PR from the user's reference or current branch;
  resolve ambiguity before merging. Record its URL, base and head repositories
  and branches, exact head SHA, and its own commit count, excluding the rest
  of a stack or an unrelated local branch.
- Stop for a draft or closed, unmerged PR. For an already merged PR, use its
  recorded result and proceed to [verification and cleanup](#verify-synchronize-and-report),
  skipping merge-only readiness, eligibility, and intent checks.
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
Outside a native stack, the trunk is the base repository's default branch
unless instructions establish another.

Always search this PR's head repository for open dependent PRs, including
forks and all pages; verify their `base.repo` and `base.ref` match its head
repository and ref. Check local stack catalogs and identified local dependents;
record dependent PRs' base refs and head SHAs before merging.

Read [Stacked PRs](references/stacked-prs.md) before mutation if a native or local
stack is found, the PR's base differs from the trunk, dependencies are found,
or stack evidence is uncertain. Follow its predecessor discovery and eligibility
rules. API errors are not evidence of absence; resolve uncertain membership
before merging when stack evidence exists.

## Confirm the merge intent

For an unmerged PR, evaluate destination and squash intent independently
before mutation. For native stacks, establish destination and eligibility
using the stack reference first.

| Confirmation needed when | Action |
| --- | --- |
| The actual destination differs from the base repository's default branch and has not already been confirmed. | Name both branches, explain the possible stack, and confirm the target, even when this skill was explicitly invoked. |
| This skill was selected implicitly, the PR has multiple commits, and squash intent has not been established. | State the count and ask to combine the commits. |

An explicit squash request, invocation of `$gh-pr-merge-squash`, a direct request
to use this named skill, or an applicable standing instruction to squash by
default establishes squash intent. A generic "merge this PR" alone confirms
neither a non-default destination nor squash intent. Combine applicable questions
and wait for the answer; do not repeat an established confirmation. Do not
retarget the PR or merge dependencies instead.

## Merge on GitHub

Prefer GitHub's squash merge without first rewriting the topic branch. Do not
rebase just for linear history. If rules, conflicts, or requested validation
require an update, recheck the head afterward and use a lease against the
inspected head for any necessary force-push. Before any worktree update,
read and apply [local file protection](references/local-cleanup.md#protect-local-files).

Wait for applicable pending checks for up to five minutes before submission,
unless the user specifies another duration. Poll with backoff from five to
thirty seconds. The window starts at the first check wait in this run;
status or head changes do not restart it. At the deadline, refresh readiness
once. If still pending or unverifiable, report the PR URL and latest check
states; stop without submitting or cleaning up branches.

Report failures, outstanding review requirements, conflicts, or unavailable
readiness information; do not bypass them or use administrator overrides.
Report out-of-scope blockers rather than expanding the merge request.

Respect a configured merge queue: its method takes precedence over `--squash`.
Before auto-merge or enqueueing, verify it satisfies the single-commit
requirement; stop if incompatible or unverifiable.

Choose a meaningful subject from the final diff and PR under repository commit
conventions, not a blind concatenation of intermediate messages. For direct
merges, supply the subject and body explicitly; use an empty body unless extra
context is useful.

The single-commit requirement and the commit-message requirements are separate.
Sending an explicit subject and body, including an empty string, does not by
itself establish that GitHub will use that exact message. Before submission,
check the selected merge path's message controls and relevant repository
defaults; for native stacks, apply the reference's message limitations.

If the path may generate, append, or replace message text, explain the concrete
limitation before merging or enabling auto-merge/queueing. Distinguish the
message requested from what can be guaranteed. If the user or repository rules
require an exact subject or body, including a strictly empty body, stop before
submission unless the path is known to honor that requirement. Do not silently
relax it. Otherwise the skill's chosen wording and empty-body default are
preferences: disclose any known limitation and continue the authorized merge
without another confirmation.

Keep the GitHub merge workflow. Do not rewrite branches or change repository
settings merely to work around message limitations.

Immediately before every submission, refresh the PR and verify the checked
head SHA, target repository/branch, and stack membership/order. If changed,
refresh scope and readiness, re-evaluate squash intent, and reconfirm the target
when required. For stacks, recheck eligibility; stop if another unmerged PR
would be included.

For an ordinary PR:

```sh
gh pr merge "$pr_url" --squash --match-head-commit "$head_sha" \
  --subject "$subject" --body ""
```

For a useful body, use `--body-file "$body_file"` with an agent-written body.
For native stacks, use the [asynchronous merge procedure](references/stacked-prs.md#submit-and-track-the-merge)
directly. Do not pass `--delete-branch`: remote deletion requires an explicit
request or the repository's automatic policy.

After a failed or uncertain attempt, read the PR state before retrying; never
retry an unchanged blocker. For asynchronous, queued, auto-merge, or resumed
requests, follow [Wait for merge completion](references/merge-completion.md)
before observation or retry.

## Verify, synchronize, and report

Once GitHub confirms the merge, including for already merged PRs, follow
[local verification and cleanup](references/local-cleanup.md). It includes
fresh dependency discovery and routes to stack reconciliation when needed.

Report the PR URL, resulting commit, local target status, and whether the topic
branch was deleted or retained and why. Distinguish merge success from incomplete
cleanup; never hide errors. Report actual commit-message discrepancies and
dependent PR retargeting or remote head rewrites, including local branches
remaining on old history and needing stack synchronization.
