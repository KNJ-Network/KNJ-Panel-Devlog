# Phase 27 - Handling a Batch of Messages at Once

v0.5.0: multi-select. A checkbox on every message row, a select-all in the header, and a toolbar
that appears once anything's checked with a folder picker and Move/Delete — so clearing out a batch
of newsletters or filing a run of receipts is one action instead of opening each message.

Move and Delete share the same form, but only Delete should ask for confirmation, since Move is
easily reversible. The in-app confirm modal was generalized so one form can have one action that
confirms and one that doesn't.

Verified by hand: three real messages, two selected, bulk-moved into a scratch folder created for
the test, then moved back — the unselected message untouched throughout.
