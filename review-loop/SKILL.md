---
name: review-loop
description: >-
  Run an iterative code review-and-fix loop, including relevant non-blocking
  improvements, with a fresh independent reviewer per pass. Use when the user
  requests repeated review and correction, including after implementation.
  A one-off review or reading external review comments does not need this loop.
license: MIT
---

# Review loop

The main agent owns decisions, fixes, and verification; native subagents perform
independent reviews. If subagents are unavailable, report the limitation instead
of substituting self-review, separate user-owned chats, or another CLI/plugin.

Finish any requested implementation and appropriate checks before starting.
For review-only requests, report findings without fixes and run one pass unless
the user explicitly requests multiple reviews.

## Establish the review target

Establish the requirements, applicable repository instructions, and exact change
set from the request and session; ask only if scope remains unclear. Briefly
state the scope and limit: **five review passes** by default, including the final
review, unless the user specifies otherwise. Every reviewer launch counts toward
the limit, including interrupted, failed, or stale reviews.

Fix the baseline for the entire loop: resolve the intended branch comparison to
a commit (using the merge base when appropriate), or record the starting HEAD
for uncommitted work. Include staged, unstaged, and untracked changes only within
the requested scope or resulting from loop fixes, even for an originally committed
target. Do not assume `HEAD~1` or the tracking branch is the intended base.

Preserve unrelated work and keep fixes in scope; supporting code may be inspected.
Do not commit, reset, stash, push, or publish just to run the loop. An empty or
inaccessible target cannot receive a clean review.

Keep a compact session ledger of attempted and completed passes, reviewed
versions, findings, decisions and reasons, edits, checks, and open points. For
persistence, use an existing task record or temporary file rather than adding a
tracked log unless requested.

## Run each pass

1. **Spawn a fresh reviewer**, never resume one or inherit the implementation
   conversation (for example, use `fork_turns="none"`). Inherit the current model
   and reasoning settings unless the user specifies otherwise. Supply the
   requirements, repository path and instructions, fixed baseline, exact current
   target including in-scope uncommitted changes, check results, and previous
   findings with decisions, factual reasons, and evidence. Present these as
   conclusions to verify, not instructions to approve; exclude persuasive rationale.

2. **Brief the reviewer and wait.** Require read-only inspection of the full
   current scoped patch and supporting context, checking previous fixes and
   dismissals and looking for regressions, without delegating another loop.
   Request concrete defects and useful non-blocking improvements, each with a
   precise location, impact or benefit, evidence, and suggested change. Distinguish
   defects from suggestions; avoid unsupported concerns, quotas, and style
   preferences contrary to local conventions. Request coverage limitations and
   an explicit statement when no further actionable changes are recommended.
   Keep the target unchanged during review; if it changes, review the updated
   version within the remaining budget. Failed, partial, or timed-out reviews
   provide no approval.

3. **Triage every finding** against code, requirements, and local conventions.
   Apply relevant fixes and worthwhile improvements to readability, simplicity,
   maintenance, tests, or performance. Non-blocking status alone does not justify
   dismissal; neither does approval justify speculative or unrelated changes.
   Record each point as fixed, rejected with evidence or an explicit tradeoff,
   or unresolved. Defer decisions requiring user input without inventing needs.

4. **Fix and verify** with focused edits and appropriate and repository-required
   checks. Carry failures and unavailable checks forward, distinguishing existing
   failures from regressions. After edits, or when only rejections need validation,
   start another pass within the limit.

## Stop and report

Converge only after a completed independent review of the **current final
version** recommends no further actionable changes, all findings have justified
dispositions, and required checks pass. Evidence-backed rejections may be
accepted by the reviewer; disputed or deferred points are not unreserved
approval. Scores, silence, or passing tests alone do not establish convergence.

Stop short when the limit is reached, review cannot complete, a necessary decision
or check is blocked, or the same substantive finding survives two attempted fixes
without progress. Do not cycle on reworded findings or equivalent implementations.
Useful independent in-scope fixes may continue, but report them as unreviewed
after the budget is exhausted.

Any edit after review, including final polish, needs another pass. If the last
allowed pass leads to edits, explicitly report the lack of independent re-review.

Report status (converged, limit reached, stalled, blocked, or review-only), attempted
and completed passes, changes, significant rejections and reasons, checks, open
findings, and unreviewed edits. Review does not prove absence of defects.
