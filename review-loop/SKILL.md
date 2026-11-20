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
launch. Finish any requested implementation and run relevant checks before the
first pass. Stop if the review target is empty or inaccessible.

For a patch review, pin the comparison base for the whole loop. Use the initial
`HEAD` when reviewing only uncommitted Git changes. Review all changes against
that base together, including in-scope untracked files. Review the index
separately when staging is in scope. For a content review, include every
requested file. Include loop fixes in subsequent passes and preserve unrelated
work and staging.

## Review, fix, and verify

1. Start a fresh subagent conversation. Provide the repository path and
   instructions, requirements, full target, comparison base if applicable, check
   results, and relevant prior decisions. Ask for a read-only review covering
   defects and worthwhile in-scope suggestions. Request locations, evidence,
   coverage limits, and a verdict.
2. Keep the target stable and discard the verdict if it changed during review.
   Evaluate findings against the requirements. Record the reasons for
   non-obvious decisions and any findings still open.
3. Apply focused fixes and worthwhile in-scope improvements. Rerun relevant
   checks, and use another pass to review changes or disputed decisions within
   the limit.

## Stop and report

The loop converges when an independent review of the **final version** finds no
actionable changes, all findings are addressed or explained, and required checks
pass. Stop reviewing at the pass limit or when further passes cannot make
progress. Finish justified fixes and checks, then report the outcome, passes
used, check results, open findings, and any edits since the last independent
review.
