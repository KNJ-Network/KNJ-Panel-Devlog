# Phase 178 - Closing the Loop on Two Patterns

Two bugs this week turned out to be patterns, so we checked for the same shape elsewhere.

## The log that doubled itself

The malware scan log showed each line twice because output was appended by a live callback and again from the final result. The cPanel restore importer had the same pattern. Both now use the callback for live progress and take the final log from the result once.

## The stale relation

The missing-DNS-records fix traced back to data cached before new records were added. Two more functions in the same area looked similar. One had a real bug: zone-file import could insert two identical records from the same batch. It now tracks records as they are created. The other had no live bug, so we added a comment explaining why it is safe now and what would change that, rather than rewriting working code.

## Verifying

Regression tests cover both fixes: the log appears once, and a duplicate within one import creates one record. 2694/2694 passing.

Tested (2694/2694).
