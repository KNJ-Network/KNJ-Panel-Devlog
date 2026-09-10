# Phase 220 - The State Nobody Ever Entered

Somewhere in this system there was already a pill for exactly this moment. Amber,
a spinning icon, the label "Issuing…" — built, styled, wired into the same component every
other status pill on the page already used. It had never once been seen doing its job. Not
because it didn't work. Because nothing ever told it to.

## A state that existed and a state that happened

Every enum in this codebase describing a certificate's status has always had a real "in
progress" value sitting right next to "active" and "failed" — not a gap that needed filling,
a state someone had already thought through and given its own color. The bug wasn't a missing
feature. It was a brand new record starting life with none of its columns actually touched by
the one line of code that was supposed to say "this is happening now." It defaulted, silently,
to the value that means nothing has been attempted — the same value a domain sitting completely
untouched for a year would also show. Two domains in two entirely different situations, wearing
the identical badge.

## The fix that already existed, right next to the bug

The strange part is how close the correct version already lived. The exact two lines needed —
mark it in-progress, *then* hand it to the background job — were already written, already
shipped, already working, on two other paths through this same feature. A retry button already
did it right. An admin-side action already did it right, with a comment explaining exactly why
the order mattered. The bug wasn't a hard problem nobody had solved. It was two newer entry
points that quietly skipped a step every older one had already learned to include, and nothing
enforced that they couldn't.

## Fixing the second complaint by not inventing anything

The other half of the report was the page never updating on its own — reasonable, on its own,
to reach for something clever: watch the state, patch just the one piece of the page that
changed, no visible refresh at all. But a plainer answer was already sitting one page over,
already shipped, already proven against exactly this same waiting-on-a-background-job shape.
Reusing it instead of building a nicer version meant one page's habits stayed predictable next
to the other's, for a fraction of the new code, and nothing about the actual complaint — *tell
me when it's not stuck, without me pressing a button* — asked for more than that.

## What was actually missing

Not a status. Not a color. Not a refresh mechanism. Two lines, at the start of two functions,
setting a value to the state that already meant "in progress" before handing the rest of the
work to something that runs later and might take a moment to get there. Everything downstream
of that line was already correct, already waiting, already built for exactly this — it had
just never been given the chance to run.
