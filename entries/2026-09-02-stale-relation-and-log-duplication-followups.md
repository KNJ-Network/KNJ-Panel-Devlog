# Phase 178 - Closing the Loop on Two Patterns

Two bugs this session turned out not to be one-off mistakes — they were a shape of bug, and once a
shape is visible in one place, the honest next step is checking everywhere else it could be hiding.

## The log that doubled itself, found in a second place

The malware scanner's status page was showing every line of its scan log twice — a callback streaming
live progress into the run's log, and the code that ran after the process finished adding the same
complete output back on top of it, since Symfony's `Process` buffers the full combined output
internally whether or not a callback is also attached. That fix (Phase 174) used the exact same
pattern the cPanel-restore importer had already been using for months: a streaming callback, followed
by an unconditional re-append of `$result->output()`. Same shape, same bug, just never actually
watched closely enough to notice on a real import — a restore's log is long and dense with real
content, and a doubled line buries itself in the noise a lot more easily than a scanner's one clean
"OK" does.

Both `runImport()` and `runImportIntoAccount()` had it. Both are fixed the same way: the streaming
callback now exists purely to push live progress to the run's log while it's still running, and the
final, authoritative log comes from the process result once, combined with whatever real log content
already existed before the process started (per-database restore lines, username-mismatch warnings) —
not by continuing to accumulate on top of what the callback already streamed.

## The stale relation, found in a second real bug

The missing-DNS-records fix (Phase 176) traced back to `$zone->records` — a cached relationship
collection that goes stale the moment something creates a new record on the same `$zone` object after
it's already been loaded once. That fix touched one function. Two more in the same file used the
identical pattern, and checking both mattered: one turned out to have a real, live bug of its own, the
other genuinely didn't.

`importRecordsLocked()` — the code path behind importing a parsed zone file — checked for duplicates
against that same stale collection, in a loop that could create several new records in a single call.
A batch containing two identical entries never caught the second one as a duplicate of the first; it
just inserted both, silently doubling a record that should have been rejected. Fixed by maintaining a
real, mutable snapshot of the zone's records that gets updated after every create, so a duplicate
within the same import is caught exactly like one that was already there.

`restoreDefaultRecordsLocked()` uses the same stale collection just as freely, and it's tempting to
"fix" it the same way on principle. Reading it closely first mattered here: every single check in that
function queries for a distinct record type-and-name pair that no earlier step in the same call ever
creates, so there's no live bug to point to — today. Rewriting a working function's data access on a
hunch, with no failing case to show for it, isn't a fix, it's scope creep with a bug-fix costume on. It
got a comment instead: exactly why it's safe right now, and exactly what would make it not be, so the
next person to extend this function doesn't have to rediscover the trap from scratch.

## Verifying the difference between "fixed" and "should be fine"

Regression tests for both real fixes: the import-log tests assert the process's own output string
appears in the final log exactly once, not twice; the duplicate-batch test imports two identical
records in one call and confirms exactly one gets created, not two. No test was written pretending to
verify the `restoreDefaultRecordsLocked()` non-fix, because there's no behavior change to verify —
the existing suite already covers what that function actually does today. 2694/2694 passing, full
suite included.

Tested (2694/2694).
