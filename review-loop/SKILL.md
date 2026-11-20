---
name: review-loop
description: >-
  Run independent review-and-fix passes with native subagents when the user
  requests iterative review and correction.
license: MIT
---

# Review loop

The main agent makes decisions, applies fixes, and runs checks. If native
subagents are unavailable, report that the independent review is blocked.

## Set the target

State the scope and pass limit, **five** by default. Count every reviewer
launch, including incomplete or stale passes.

For a patch review, fix the comparison base for the loop. In Git, resolve the
intended branch comparison to a commit or use the starting HEAD for worktree and
index changes alone. Include in-scope committed, staged, unstaged, and untracked
changes. Inspect the staged and unstaged layers separately so one cannot hide
the other.

For a content review, include all requested files, even if they are unchanged.
Include loop fixes in either mode and preserve unrelated work and staging.
Finish any requested initial implementation and run relevant checks before the
first pass.

## Review, fix, and verify

1. **Spawn a fresh reviewer.** Give a native subagent the requirements,
   repository path and instructions, comparison base if applicable, current
   target, check results, and relevant prior decisions with their evidence.
   Start a fresh conversation where supported (for example,
   `fork_turns="none"`). Ask for a read-only review of the full target,
   including prior fixes and dismissals. Request defects and worthwhile
   suggestions with locations and evidence, plus any coverage limits and an
   explicit verdict.
2. **Triage the findings.** Keep the target stable during review and check that
   it still matches when the reviewer returns. If it changed, discard the stale
   verdict and review the new target in a remaining pass. Check findings against
   the requirements and conventions. Record why you accept, dismiss, or defer
   each one.
3. **Fix and verify.** Apply focused fixes and worthwhile in-scope improvements.
   Rerun relevant checks. Use another pass to review changes or verify disputed
   decisions, within the limit.

## Stop and report

Converge when a completed independent review of the **final version** finds no
actionable changes, all findings have justified dispositions, and required
checks pass.

Stop if the target is empty or inaccessible, the pass limit is reached, or
further passes are blocked or keep raising the same issue without progress.
Finish justified fixes and checks even if no review passes remain. Report the
outcome, passes used, checks, open findings, and any edits made after the final
independent review.
