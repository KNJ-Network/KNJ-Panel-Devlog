# Phase 27 - Handling a Batch of Messages at Once

v0.5.0: multi-select. A checkbox on every message row, a select-all in the header, and a
toolbar that appears the moment anything's checked with a folder picker and Move/Delete —
so clearing out a batch of newsletters or filing a run of receipts is one action instead of
opening each message individually.

The small technical piece worth noting: Move and Delete share the same form, but only Delete
should ask for confirmation — Move is trivially reversible, Delete (when it's actually
permanent, in Trash) isn't. The in-app confirm modal built a couple of phases back only knew
how to read a confirmation message off the form itself, which would have meant either both
buttons prompt or neither does. Generalized it to also check the specific button that was
clicked, so one form can have one action that confirms and one that doesn't.

Verified by hand: three real messages, two selected via checkbox and select-all both, bulk-
moved into a scratch folder created for the test, then bulk-moved back — the third,
unselected message untouched throughout, exactly as it should be.
