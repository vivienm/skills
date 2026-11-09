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

Finish the requested implementation and checks first. The main agent owns
decisions, fixes, and verification. Use native subagents for independent
reviews. If they are unavailable, report the limitation and stop the loop.

## Establish the review target

Identify the requirements, repository instructions, and exact change set. State
the scope and pass limit, **five passes** by default. Every reviewer launch
counts, including the final review and failed, interrupted, or stale reviews.

Keep one baseline commit throughout. For a branch comparison, resolve the
intended base, using the merge base when appropriate. For uncommitted work,
record the starting HEAD. Include all in-scope staged, unstaged, and untracked
changes and subsequent fixes, even for an originally committed target.

Preserve unrelated work. Do not commit, reset, stash, push, or publish just to
run the loop. An empty or inaccessible target cannot receive a clean review.

Record attempted and completed passes, target versions, findings, decisions,
fixes, checks, and open points in the task record or a temporary file.

## Run each pass

1. **Spawn a fresh reviewer.** Supply the requirements, repository path and
   instructions, fixed baseline, exact current target, check results, and
   relevant prior decisions. Present earlier conclusions neutrally for
   verification. Start without the implementation conversation (for example,
   `fork_turns="none"`) and inherit model and reasoning settings unless the user
   specifies otherwise.

   Request read-only review of the full scoped patch and supporting context,
   including prior fixes, dismissals, and possible regressions. The reviewer
   performs a single pass. Ask for defects and useful non-blocking suggestions
   separately, with locations, evidence, impact or benefit, and proposed
   changes. Ask for coverage limitations and an explicit verdict when no
   actionable changes remain. Keep the target fixed during the pass. A failed,
   partial, timed-out, or stale review cannot establish convergence.

2. **Triage findings.** Check each against the code, requirements, and local
   conventions. Accept relevant fixes and worthwhile improvements. Record
   whether each finding is accepted, rejected, or unresolved, with the evidence
   or tradeoff behind the decision.

3. **Fix and verify.** Make focused edits and rerun relevant checks. Record
   failed or unavailable checks, distinguishing existing failures from
   regressions. Review again within the limit if the target changed or a
   rejection needs validation.

## Stop and report

Converge only when an independent review of the **final version** finds no
actionable changes, all findings are resolved, and required checks pass. The
reviewer may accept evidence-backed rejections. Report edits outside the last
completed review as independently unreviewed.

Retry transient reviewer failures within the pass limit. Stop the loop at the
limit, on a persistent reviewer failure or blocked decision or check, or when
two attempts to fix the same finding make no progress. Useful in-scope fixes may
continue after the limit.

Report status (converged, limit reached, stalled, or blocked), checks, remaining
issues, significant decisions and reasons, and independently unreviewed edits.
