# Phase 208 - Completed Isn't the Same as Nothing To Do

A server had six real packages sitting there, ready to install. Its own row said "Completed."
Not wrong, exactly — the last thing that had happened to it really had finished successfully. Just
not the question anyone looking at that column actually wanted answered.

## The status column was answering a different question than it looked like

"Completed" described one specific run, whenever it happened to be, however long ago. It said
nothing about right now. A server nobody had ever touched read exactly the same as one currently
caught up, because both fell back to the same quiet, blank state. The fix wasn't teaching the
column a new fact — the fact already existed one column over, as a plain number. It just needed to
outrank the old answer whenever the two disagreed: a server sitting on real pending packages, not
mid-update, now says so directly, no matter what its last run happened to report.

## The number that stopped updating itself

A second, quieter problem showed up right after the first fix: a server that had just genuinely
finished catching up still showed its old, pre-update count. The cache behind that number had
already been cleared the moment the update finished — clearing it just left nothing there until
someone else happened to ask again, which in practice meant a manual click or an unrelated page
reload. Refreshing it the moment the number goes stale, rather than waiting for the next visitor to
notice and re-ask, closed that gap for the server sitting right here. A linked server elsewhere
needed the same idea taught to the part of the system watching *its* run finish — the same
principle, just needing to fire from a different place.

## The bug that only existed in the gap between two writes

Wiring that in surfaced something stranger: the fresh number and the "it's done" signal were
occasionally telling two different stories. Not because either one was wrong on its own — because
they finished at slightly different moments, and something reading state in between could catch
the first without the second. The known-good fix wasn't a retry or a longer wait. It was ordering:
finish refreshing the real number *before* announcing that anything is done, so nothing watching can
ever observe one without the other. A one-line reorder, and a genuinely interesting kind of bug —
one a test run alone, however faithful, was never actually positioned to expose. It only shows up by
watching something real happen live, at the exact moment two separate pieces of work are racing to
finish first.
