# Wait for merge completion

Read this reference when a merge is asynchronous, queued, auto-merge-enabled,
or when resuming observation of an existing request.

Queued or auto-merge-enabled is not yet merged. Unless the user specifies
another duration, observe completion for up to five minutes after submission,
polling with backoff from five to at most thirty seconds between requests.
Use one observation window across asynchronous processing and queue waits;
a status change does not restart it. For an existing request, start the
window when resuming observation.

At the deadline, refresh the PR state once and let that result determine
the next step. If merged, continue with verification, synchronization, and
guarded cleanup, even if the last asynchronous result was `pending`. If closed
without merging, report that state and defer cleanup. If still pending,
report the PR URL, latest state, and asynchronous request UUID when available,
and defer cleanup. Explain that the server-side operation may still complete;
ending observation does not cancel it. If the final state cannot be retrieved,
report the uncertainty and defer cleanup. On resumption, inspect the existing
request and PR state before considering a new submission.
