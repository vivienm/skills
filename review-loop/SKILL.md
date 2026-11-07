---
name: review-loop
description: >-
  Run independent review-and-fix passes until convergence or a pass limit,
  including useful non-blocking improvements. Use for requested iterative review
  and correction, not one-off reviews or reading external review comments.
  Requires native subagents.
license: MIT
---

# Review loop

The main agent owns decisions, fixes, and verification; native subagents review
independently. If unavailable, report the limitation instead of substituting
self-review, separate user-owned chats, or another CLI/plugin.

Finish any requested implementation and appropriate checks before starting.

## Establish the review target

Establish requirements, repository instructions, and the exact change set; ask
only if scope remains unclear. State the scope and pass limit: **five passes**
by default, including the final review. Every reviewer launch counts, including
interrupted, failed, or stale reviews.

Fix the baseline for the entire loop: resolve the intended branch comparison to
a commit (using the merge base when appropriate), or record the starting HEAD
for uncommitted work. Do not assume `HEAD~1` or the tracking branch is the base.
Include in-scope staged, unstaged, and untracked changes and loop fixes, even
for an originally committed target.

Preserve unrelated work; supporting code may be inspected. Do not commit, reset,
stash, push, or publish just to run the loop. An empty or inaccessible target
cannot receive a clean review.

Keep a compact ledger of attempted and completed passes, reviewed versions,
findings and decisions, edits, checks, and open points. Use an existing task
record or temporary file; do not add a tracked log unless requested.

## Run each pass

1. **Brief and spawn a fresh reviewer.** Include requirements, repository path
   and instructions, fixed baseline, exact current target including in-scope
   uncommitted changes, check results, and relevant prior ledger entries.
   Present previous conclusions for verification, without persuasive rationale.

   Require read-only inspection of the full scoped patch and supporting context,
   including previous fixes, dismissals, and possible regressions; do not
   delegate another loop. Request defects and useful non-blocking suggestions
   separately, with locations, impact or benefit, evidence, and suggested
   changes. Exclude unsupported concerns, quotas, and preferences contrary to
   local conventions. Ask for coverage limitations and an explicit statement
   when no actionable changes remain.

   Launch with the complete brief and no implementation conversation (for
   example, `fork_turns="none"`); never resume a reviewer. Inherit current model
   and reasoning settings unless the user specifies otherwise. Keep the target
   unchanged while waiting; if it changes, review the new version within the
   remaining budget. Failed, partial, or timed-out reviews provide no approval.

2. **Triage every finding** against code, requirements, and local conventions.
   Accept relevant fixes and worthwhile improvements; non-blocking status alone
   does not justify dismissal. Avoid speculative or unrelated changes. Record
   each decision and its reason: accepted, rejected, or unresolved. Support
   rejections with evidence or an explicit tradeoff. Defer decisions requiring
   user input without inventing needs.

3. **Fix and verify** accepted findings with focused edits and required or
   otherwise relevant checks. Record fixes and carry failed or unavailable
   checks forward, distinguishing existing failures from regressions. Start
   another pass within the limit after edits or when rejections need validation.

## Stop and report

Converge only when a completed independent review of the **current final
version** recommends no further actionable changes, all findings have justified
dispositions, and required checks pass. The reviewer may accept evidence-backed
rejections; disputed or deferred points are not unreserved approval. Scores,
silence, or passing tests alone do not establish convergence. Any edit after
review, including polish, needs another pass; otherwise report it as
independently unreviewed.

Retry transient reviewer failures within the pass limit. Stop when the limit is
reached, a reviewer failure cannot be resolved, a necessary decision or check is
blocked, or the same substantive finding survives two attempted fixes without
progress. Do not cycle on reworded findings or equivalent implementations.
Useful in-scope fixes may continue after the budget is exhausted.

Report status (converged, limit reached, stalled, or blocked), checks, remaining
issues, significant decisions and reasons, and independently unreviewed edits.
