# Phase 234 - The Badge's Quieter Sibling

Having just found one number that was doing too much work, the obvious next question wasn't
whether it had been fixed. It was whether it had been alone.

## The same question, asked of the rest of the house

A bug like the one just closed rarely has a single occurrence. The shape of it — a cheap-looking
number that's secretly allowed to go ask a slow, external thing a live question — isn't specific to
package updates. It's a pattern, and once a pattern's been named, the honest next step is checking
whether anything else in the building follows it too, not assuming the one instance found was the
only one that existed.

## A quieter version of the same mistake

It was. A card on the same dashboard, showing whether mail is actually running, had been built to
answer a slightly different question — not "what packages need updating" but "is this service alive
right now" — and inherited the same shape of risk by following the same instinct: if you don't know
the answer, go find out, right now, on whoever happened to ask. When mail runs on a separate machine
reached over the network, "go find out" means a live round trip to that machine, and if that machine
is ever slow to answer, whoever asked pays for it.

The two didn't look alike on the surface. One waits on a package mirror for minutes at the outside.
This one waits on a linked server for half a minute. But the wait itself was never the point — the
point was who was allowed to be the one who triggers it. A dashboard glancing at a status is not a
request for a fresh answer. It never was. It just hadn't been told that yet, the same way the number
it sits next to hadn't been told that either, until a few hours earlier.

## Recognizing the shape, not just the symptom

Because the underlying mistake had just been named clearly, this one wasn't found by getting caught
in the act the way the first was. It was found by asking a more general question of the code itself:
*does anything else assume a page load is a good moment to go ask something slow for a live answer?*
One other place did.

## What actually finished

The dashboard's glance at mail status is a glance again, not a live poll pretending to be one —
reading whatever the last real check found, never running one of its own. The place that's meant to
ask for a fresh answer still does, exactly where asking is the entire reason someone opened it. Same
fix, same shape, applied on purpose the second time instead of by accident.
