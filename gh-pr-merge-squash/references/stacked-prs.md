# Stacked PRs

A native GitHub stack, an ordinary chain of PR base branches, and a local
`gh-stack` catalog are distinct; inspect the relevant sources separately.

## Discover the remote scope

Use the verified host; pass `--hostname` explicitly outside GitHub.com. Here
`repo` is the PR's base repository (`OWNER/REPO`) and `pr_number` its PR number,
never a stack number. For dependency searches, follow the repository identity
and pagination rules in [initial discovery](../SKILL.md#detect-stacks-and-dependents).

```sh
gh api "repos/$repo/pulls/$pr_number"
gh api --method GET "repos/$repo/stacks" -f pull_request="$pr_number"
```

Read the PR's `stack` metadata and, when present, retrieve the full stack
with `GET /repos/{owner}/{repo}/stacks/{stack_number}`. Use the filtered list
when membership is unclear. Record the stack base and ordered PRs with their
states, head repositories/refs/SHAs, and immediate bases. Apply initial
discovery's uncertainty rule to unsupported endpoints as well as API errors.

Follow further dependent PRs when relevant.

## Qualify predecessor candidates

For predecessor traversal, the trunk is the native stack's declared base,
otherwise the base repository's default branch unless instructions establish
another trunk. When the PR's base differs from the trunk, follow unmerged
predecessor candidates whose head repository/ref matches each successive base;
include closed, unmerged PRs. Stop at the trunk even if PRs originate there.

A matching head repository/ref is only a candidate: a closed PR may predate
branch deletion and recreation. Compare its recorded head SHA with the fetched
current base tip and history, using explicit stack or local catalog membership
when available.

Exclude unrelated historical PRs, but do not infer that a PR is unrelated from
a failed ancestry check alone: history may have been rewritten. Resolve
ambiguities before merging; neither a name match nor uncertainty establishes
or rules out a dependency.

## Establish destination and eligibility

For a native stack, the actual landing destination is the stack base, which
may differ from the PR's immediate base. When target confirmation is required
for an eligible PR, explain both destinations.

For both native stacks and ordinary PR chains, this skill permits merging
only the lowest unmerged PR: every PR below it must be confirmed merged.
Closed without merging, queued, pending, or unknown does not qualify. If any
predecessor is not confirmed merged, stop before mutation and name the lowest
unmerged PR to handle first. Do not offer confirmation to bypass this rule or
merge a predecessor on the user's behalf; leave upper PRs unmerged.

GitHub's native merge includes every open PR below the selected one. Never
submit a multi-PR batch or whole-stack target, or rely on ambiguous numeric
resolution in `gh stack merge`.

If the user explicitly requests a sequence of PR merges, process only the
current lowest unmerged PR, wait for its confirmed merge, then refresh scope
and eligibility for the next. Apply SHA and message controls per PR and
re-evaluate checks and reviews after automatic head rewrites. Squash means
one commit per PR.

## Submit and track the merge

After the main skill's readiness and confirmation checks, use GitHub's
asynchronous endpoint directly; do not first try `gh pr merge`, which does not
support native stacks. Avoid `gh stack merge` unless its installed version
can unambiguously target the PR and bind the checked SHA and explicit message.
The main skill's final scope refresh must include the remote stack and
eligibility: the payload's `sha` guards only the selected PR head.

With no merge queue configured for the destination, explicitly select a direct
squash merge and supply the message; use a file for a nonempty body:

```sh
jq -n --arg sha "$head_sha" --arg title "$subject" \
  '{sha: $sha, merge_method: "squash", merge_action: "direct_merge",
    commit_title: $title, commit_message: "", bypass_rules: false}' \
  > "$payload_file"
gh api --method PUT "repos/$repo/pulls/$pr_number/merge-async" \
  --input "$payload_file"
```

With a configured queue, apply the main skill's single-commit and exact-message
requirements. Use `merge_action: "merge_queue"`, omitting the direct-merge-only
fields `merge_method`, `commit_title`, and `commit_message`. The queue controls
method and message; do not claim an explicit message was supplied. The `default`
action also selects a configured queue even if it is optional, discarding
direct-merge controls. Resolve unknown queue configuration before submitting;
never use `direct_merge` to evade a queue or enable rule bypass.

Read [Wait for merge completion](merge-completion.md) for the shared observation
window, backoff, deadline, and resumption rules. An immediate `200` can report
`merged` or `enqueued` without a UUID; apply the status rules below before
attempting UUID polling. For a pending request, keep its UUID and poll with:

```sh
gh api "repos/$repo/pulls/$pr_number/merge-async/$uuid"
```

- `202` means accepted, not merged. Rules are evaluated during execution.
- `pending`: keep polling within the observation window. Until confirmed merged,
  do not start another merge or clean up branches.
- `merged`: use the result SHA for [local verification and cleanup](local-cleanup.md),
  which still requires confirming the PR's merged state.
- `enqueued` is terminal for this asynchronous request, but the PR has not
  merged. Follow the PR/queue state for completion, not the same UUID, using
  the remaining observation window.
- `failed`: report the reason and apply the main skill's retry rules.
- `409` can identify an already pending request. Inspect its UUID, SHA and
  options rather than resubmitting; report any mismatch with the intended
  operation. After a timeout or expired UUID, recover from current PR state.

An empty `commit_message` can still yield a generated body; local verification
must inspect the actual message.

## Reconcile dependents and retire the merged branch

During [pre-deletion discovery](local-cleanup.md#refresh-dependencies-before-deletion),
refresh the stack and every recorded dependent PR, including those now closed.
Compare states, base refs, and head SHAs with the snapshot: GitHub may retarget
and rebase surviving PRs. Fetch current heads without resetting local branches;
report differing local/remote tips and any required synchronization. Never
merge or force-push a stale local child onto rewritten remote history.

In addition to [local cleanup](local-cleanup.md), deletion requires:

- No open dependent PR still targets that branch. If the transition is
  pending or cannot be verified, retain it with that concrete reason.
- No active local stack still needs the branch as an unreconciled parent.
  An existing catalog must recognize the PR as merged and surviving layers
  as reconciled with current remote heads and parents, without a pending rebase.
  Even without a catalog, retain a parent needed as the replay base of an
  identified local dependent still on old history; remote retargeting alone
  does not reconcile it. With no local stack or dependents, do not create or
  synchronize a catalog just because remote retargeting occurred.

A historical catalog entry marked merged, or past stack membership alone,
does not require retaining the branch. Delete only the merged PR's local topic
branch when all conditions hold; otherwise name the blocking dependency.

`gh stack sync` fetches, rebases, pushes, and updates metadata. Merging one PR
does not authorize rewriting/pushing survivors. Require existing synchronization
authorization, check affected worktrees, local-only commits and remote heads,
and inspect installed help to establish that checkouts preserve ignored files.
Verify resulting branches and metadata: exit code zero can accompany
`Sync aborted`.

Use `--prune` only for a known, authorized deletion set whose branches all pass
the same safeguards. Do not run broad sync/prune to delete one branch. If safe
synchronization is unauthorized or unavailable, retain the branch and report
what remains. Never delete remote refs as incidental cleanup.

## Sources

- [Repository-scoped PR listing and base/head filters](https://docs.github.com/en/rest/pulls/pulls#list-pull-requests)
- [Stack membership and API behavior](https://docs.github.com/en/pull-requests/reference/stacked-pull-requests-apis-and-webhooks)
- [Stack discovery endpoints](https://docs.github.com/en/rest/pulls/stacks)
- [Asynchronous merge request and result](https://docs.github.com/en/rest/pulls/pulls#merge-a-pull-request-asynchronously)
- [Merge order and automatic rebasing](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/merging-stacked-pull-requests)

Check current documentation and installed CLI help if these interfaces
change; do not substitute an unguarded merge command for a missing feature.
