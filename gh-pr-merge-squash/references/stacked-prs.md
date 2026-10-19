# Stacked PRs

Read this reference when the main skill's initial discovery finds a stack,
dependencies, or uncertain stack evidence. A native GitHub stack and a local
`gh-stack` catalog are separate sources of state. An ordinary chain of PR
base branches need not be a native stack.

## Discover the remote scope

Use the PR's verified host and base repository for API calls. Examples below
use `repo` for `OWNER/REPO` and `pr_number` for the requested PR, never a
stack number. On another GitHub host, pass `--hostname` explicitly.

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

Also list open PRs whose base is this PR's head ref, with pagination, and
verify repository identities; follow further dependents when relevant.
This detects ordinary dependent PRs as well as corroborating native stacks.
For an ordinary PR chain, also follow the selected PR's lower dependencies
through its base branch, resolving repository identities and PR states,
and stopping at the trunk established by the main skill. Do not search for
predecessors sourced from that trunk; a PR from the trunk to another branch
does not make it a lower dependency.
The eligibility rule below applies to native stacks and ordinary chains.

For a native stack, selecting a PR merges it and every open PR below it
into the stack base. This skill permits merging only the lowest unmerged
PR: every PR below the selected one must already be confirmed merged.
Closed without merging, queued, pending, or unknown is not merged. If this
condition fails, stop before mutation and name the lowest unmerged PR to
handle first; do not offer a confirmation that would allow a combined merge
or merge a predecessor on the user's behalf. Upper PRs must remain unmerged.
Do not use a whole-stack target or rely on ambiguous numeric resolution in
`gh stack merge`.

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

For a native stack, use GitHub's asynchronous endpoint after completing the
main skill's readiness and confirmation checks. Refresh both the requested
PR and the remote stack immediately before submission. The payload's `sha`
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

Keep the UUID from the response and poll using the main skill's
[completion observation window and backoff](../SKILL.md#wait-for-merge-completion):

```sh
gh api "repos/$repo/pulls/$pr_number/merge-async/$uuid"
```

- `202` means accepted, not merged. Rules are evaluated during execution.
- `pending` means keep polling within that observation window. At its
  deadline, follow the main skill's final PR state check: if merged, continue
  with verification and guarded cleanup; report the pending request and its
  UUID only if it remains pending. While the merge is unconfirmed, do not
  start another merge or clean up branches.
- `merged` provides a result SHA; still confirm the PR's merged state and
  verify the fetched commit as required by the main skill.
- `enqueued` is terminal for this asynchronous request, but the PR has not
  merged. Follow the PR/queue state for completion, not the same UUID, using
  the remaining observation window.
- `failed` means report the supplied reason. Re-read the PR before deciding
  whether a changed condition justifies a fresh attempt.
- `409` can identify an already pending request. Inspect its UUID, SHA and
  options rather than resubmitting; report any mismatch with the intended
  operation. After a timeout or expired UUID, recover from current PR state.

An explicitly empty `commit_message` can still yield a generated body, as
observed in the reported first use. Verify the full resulting message;
do not assume all API paths or GitHub versions behave identically.

## Reconcile dependents and retire the merged branch

After the merge, refresh the stack and every recorded dependent PR. Repeat
the paginated search for open PRs targeting the merged PR's head branch
before deletion, to discover dependents created during the wait. If this
search cannot be completed, retain the local branch. Compare current states,
base refs and remote head SHAs with the snapshot. GitHub can
automatically retarget and rebase surviving PRs. Fetch their current heads
without resetting local branches; if the remote and local tips differ,
report both and whether local synchronization remains necessary. Do not
merge or force-push a stale local child back onto the rewritten remote head.

Retire only the merged PR's local topic branch, and only after all the main
skill's identity, merged-head, worktree and ignored-file checks pass, plus:

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
these conditions hold, delete it using the main skill's guarded cleanup.
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

- [Stack membership and API behavior](https://docs.github.com/en/pull-requests/reference/stacked-pull-requests-apis-and-webhooks)
- [Stack discovery endpoints](https://docs.github.com/en/rest/pulls/stacks)
- [Asynchronous merge request and result](https://docs.github.com/en/rest/pulls/pulls#merge-a-pull-request-asynchronously)
- [Merge order and automatic rebasing](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/merging-stacked-pull-requests)

Check current documentation and installed CLI help if these interfaces
change; do not substitute an unguarded merge command for a missing feature.
