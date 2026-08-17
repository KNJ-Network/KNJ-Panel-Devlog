# Phase 94 - Fix Everything, Then Audit It

A different approach from every prior audit: go through everything first, fix whatever turns up,
and then run the audit proper. Most past audits worked the other way around (review, find, fix,
re-verify), so this one meant front-loading the review.

## Going back over already-audited code, not just what's new

Rather than reviewing only what changed since the last pass, this one covered every controller,
the whole services layer, all views and models, and the entire provisioning script, including
plenty of code that earlier audits had already passed.

That paid off directly. A scoped review of the two newest privileged file operations found the same
class of problem in both: a write into an account's home directory that didn't first check whether
the destination was a link pointing elsewhere. Rather than stop at two, the whole script was swept
for the same pattern. More instances turned up in older code, and all were fixed with one shared
path-containment check.

## The second-order bug in the "obvious" fix

Two long-lived tokens were stored in the database in plain text. The obvious fix, encrypting the
column, would have silently broken both features, because lookups by exact match can no longer work
on randomized ciphertext, and the tokens must also stay viewable on their settings pages. The fix
splits the two jobs across two columns: a deterministic hash for lookup, and the original column
kept for redisplay but now encrypted at rest. It was verified with a real round trip against the
development server's database, not just the test suite.

## What else turned up

User-chosen names that reach generated server configuration are now restricted to a safe character
set. Several places that passed sensitive values to commands now use private input channels. A few
argument-handling gaps of a class seen in an earlier audit were closed in three more places. And
the self-server record, which every account creation depends on, can no longer be edited through
the ordinary server form.

Fourteen issues in total were fixed in the same pass, and the full test suite (1,414 tests) was
re-run clean. Several higher-stakes fixes were also re-verified live on the development server. The
audit proper, run against the fixed code, found nothing further. A few items were deliberately
carried into the next phase for research before changing anything.
