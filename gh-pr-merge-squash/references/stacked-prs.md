# Stacked PRs

Read this reference when the main skill's initial discovery finds a stack,
dependencies, or uncertain stack evidence. A native GitHub stack and a local
`gh-stack` catalog are separate sources of state. An ordinary chain of PR
base branches need not be a native stack.

## Discover the remote scope

Use the verified host. For the selected PR and its native stack, the examples
below use its base repository as `repo` (`OWNER/REPO`) and its PR number as
`pr_number`, never a stack number. Dependency searches use the repositories
specified by the [initial discovery](../SKILL.md#detect-stacks-and-dependents),
including the head repository for dependents of a fork PR. On another GitHub
host, pass `--hostname` explicitly.

```sh
gh api "repos/$repo/pulls/$pr_number"
gh api --method GET "repos/$repo/stacks" -f pull_request="$pr_number"
```

Read the PR's `stack` metadata and, when present, retrieve the full stack
with `GET /repos/{owner}/{repo}/stacks/{stack_number}`. The filtered list is
also useful when membership is unclear. Record its base and ordered PRs,
their states, head repositories/refs/SHAs, and immediate bases. A local
"not in a stack" result does not override this evidence. API errors or
unsupported stack endpoints are not evidence of absence: resolve uncertain
membership before merging when stack evidence exists.

Follow further dependent PRs when relevant, using the pagination and repository
identity checks from the [initial discovery](../SKILL.md#detect-stacks-and-dependents).

## Qualify predecessor candidates

A matching head repository/ref identifies a candidate, not necessarily a
current predecessor. A closed PR may refer to an old branch that was deleted
and recreated under the same name. Compare the candidate's recorded head SHA
with the fetched current base tip and history; use explicit stack or local
catalog membership when available to establish the relationship.

Exclude unrelated historical PRs rather than blocking on their names alone.
Rewritten history can also remove ancestry, so a failed ancestry check alone
does not prove the PR is unrelated. Resolve ambiguous relationships before
merging; do not treat uncertainty as absence or automatically classify every
historical name match as an unmerged dependency.

## Establish destination and eligibility

For a native stack, the actual landing destination is the stack base, which
may differ from the PR's immediate base. When the main skill requires target
confirmation for an eligible native stack PR, explain both the immediate
base and actual stack destination. Confirmation cannot make an upper PR
eligible while a lower PR remains unmerged.

For both native stacks and ordinary PR chains, this skill permits merging
only the lowest unmerged PR: every PR below the selected one must already
be confirmed merged.
Closed without merging, queued, pending, or unknown is not merged. If this
condition fails, stop before mutation and name the lowest unmerged PR to
handle first; do not offer a confirmation that would allow a combined merge
or merge a predecessor on the user's behalf. Upper PRs must remain unmerged.

For a native stack, GitHub merges the selected PR and every open PR below it
into the stack base. Do not use a whole-stack target or rely on ambiguous
numeric resolution in `gh stack merge`.

For example, with PR1 merged, PR2 unmerged, and PR3 unmerged, PR2 is eligible
for readiness checks; PR3 is blocked by PR2. A request to merge PR3 alone
does not authorize merging PR2 first. After PR2 is actually merged, PR3 can
become eligible, subject to a fresh inspection of its state and head.

If the user explicitly requests a sequence of PR merges, process only the
current lowest unmerged PR, wait for its confirmed merge, then refresh the
stack and recheck eligibility before considering the next requested PR.
Never submit a multi-PR batch. This preserves the checked-SHA and message
controls per PR. Re-evaluate checks and reviews after every automatic head
rewrite. Squash means one commit per PR.

## Submit and track the merge

For a native stack, use GitHub's asynchronous endpoint directly after completing
the main skill's readiness and confirmation checks. Do not first try
`gh pr merge`, which does not support this merge path. Avoid `gh stack merge`
when its installed version cannot bind the operation to the checked head SHA
and explicit commit message, or its target could resolve to a stack number
instead of the intended PR. Refresh both the requested PR and the remote
stack immediately before submission. The payload's `sha`
guards only the selected PR head. Confirm again that no unmerged lower PR
would be included; a multi-PR batch is outside this skill's permitted scope.

When no merge queue is configured for the destination, create a JSON payload
with explicit message fields; use a file for a nonempty body to preserve its
text. Select a direct merge explicitly so that message and squash controls
are not silently discarded in favor of queue settings:

```sh
jq -n --arg sha "$head_sha" --arg title "$subject" \
  '{sha: $sha, merge_method: "squash", merge_action: "direct_merge",
    commit_title: $title, commit_message: "", bypass_rules: false}' \
  > "$payload_file"
gh api --method PUT "repos/$repo/pulls/$pr_number/merge-async" \
  --input "$payload_file"
```

When a queue is configured for the destination, verify its merge method and use
`merge_action: "merge_queue"`, omitting the direct-merge-only fields
`merge_method`, `commit_title`, and `commit_message`. The queue controls
method and message; do not claim that an explicit message was supplied.
If an exact requested message cannot be guaranteed, report the blocker.
The `default` action also selects a queue whenever one is configured, even
if it is not required; it does not guarantee the supplied direct-merge
method or message. If queue configuration is unknown, resolve it before
submitting. Never use `direct_merge` to evade a queue or enable rule bypass.

Read [Wait for merge completion](merge-completion.md) for the shared observation
window, backoff, deadline, and resumption rules. An immediate `200` can report
`merged` or `enqueued` without a UUID; apply the status rules below before
attempting UUID polling. For a pending request, keep its UUID and poll with:

```sh
gh api "repos/$repo/pulls/$pr_number/merge-async/$uuid"
```

- `202` means accepted, not merged. Rules are evaluated during execution.
- `pending` means keep polling within that observation window. Apply the
  completion reference's final PR state check at the deadline. While the merge
  is unconfirmed, do not start another merge or clean up branches.
- `merged` provides a result SHA; still confirm the PR's merged state and
  verify the fetched commit under [local verification and cleanup](local-cleanup.md).
- `enqueued` is terminal for this asynchronous request, but the PR has not
  merged. Follow the PR/queue state for completion, not the same UUID, using
  the remaining observation window.
- `failed` means report the supplied reason. Re-read the PR before deciding
  whether a changed condition justifies a fresh attempt.
- `409` can identify an already pending request. Inspect its UUID, SHA and
  options rather than resubmitting; report any mismatch with the intended
  operation. After a timeout or expired UUID, recover from current PR state.

An explicitly empty `commit_message` can still yield a generated body.
Verify the full resulting message; do not assume all API paths or GitHub
versions behave identically.

## Reconcile dependents and retire the merged branch

After the merge, refresh the stack and every recorded dependent PR. The
[pre-deletion discovery](local-cleanup.md#refresh-dependencies-before-deletion)
also finds dependents created during the wait. Compare current states,
base refs and remote head SHAs with the snapshot. GitHub can
automatically retarget and rebase surviving PRs. Fetch their current heads
without resetting local branches; if the remote and local tips differ,
report both and whether local synchronization remains necessary. Do not
merge or force-push a stale local child back onto the rewritten remote head.

Retire only the merged PR's local topic branch, and only after all
[local cleanup](local-cleanup.md) identity, merged-head, worktree and
ignored-file checks pass, plus:

- No open dependent PR still targets that branch. If the transition is
  pending or cannot be verified, retain it with that concrete reason.
- No active local stack still needs the branch as an unreconciled parent.
  If a local catalog exists, verify that it recognizes this PR as merged
  and that surviving layers have been reconciled with their current remote
  heads and parents, without a pending rebase. A retained historical catalog
  entry marked merged does not itself require retaining the Git ref.
  Even without a catalog, retain the parent while an identified local
  dependent branch remains on the old history and needs that parent tip
  as its replay base. Remote retargeting alone does not reconcile it.
  With no local stack or dependent local branches, completed remote
  retargeting alone does not require creating or synchronizing a catalog.

Do not retain the branch solely because it once belonged to a stack. If
these conditions hold, delete it using the local cleanup procedure.
Otherwise name the precise dependency preventing deletion.

`gh stack sync` fetches, rebases, pushes, and updates stack metadata; it is
not a read-only refresh. A request to merge one PR does not by itself
authorize rewriting/pushing surviving PRs. Run it only within existing
authorization for stack synchronization, after checking affected worktrees,
local-only commits and remote heads. Inspect the installed version's help
and establish that any checkouts preserve ignored files before running it.
Verify the resulting branches and metadata; exit code zero can accompany
`Sync aborted` and does not prove reconciliation.

Use `--prune` only if its deletion set is known and within the authorized
cleanup scope, and every branch in that set satisfies the same safeguards.
Do not run a broad sync/prune merely to delete one merged branch. If safe
synchronization is outside scope or unavailable, retain the branch and
report the remaining synchronization explicitly. Never delete remote refs
as incidental cleanup.

## Sources

- [Repository-scoped PR listing and base/head filters](https://docs.github.com/en/rest/pulls/pulls#list-pull-requests)
- [Stack membership and API behavior](https://docs.github.com/en/pull-requests/reference/stacked-pull-requests-apis-and-webhooks)
- [Stack discovery endpoints](https://docs.github.com/en/rest/pulls/stacks)
- [Asynchronous merge request and result](https://docs.github.com/en/rest/pulls/pulls#merge-a-pull-request-asynchronously)
- [Merge order and automatic rebasing](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/merging-stacked-pull-requests)

Check current documentation and installed CLI help if these interfaces
change; do not substitute an unguarded merge command for a missing feature.
