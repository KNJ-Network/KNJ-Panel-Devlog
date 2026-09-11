# Phase 225 - The List That Remembered When, Not What

Nothing about the domains list was broken. Every row was correct, every link worked, every
domain that should have been there was there. The only thing wrong with it was the order it
came in — and it turns out order alone is enough to make a perfectly working list feel
disordered.

## Sorted by the wrong thing

The list had been quietly sorting itself by one signal the whole time: when a row was
inserted. Add an addon domain, then a subdomain, then another addon domain, and that's
exactly the order they'd sit in forever after — not grouped by what they are, just laid out
in the sequence they happened to arrive in. Nothing enforced that order on purpose. It was
just what was left once nothing else was specified.

That's a fine default for a database table and a poor one for a page a person actually
reads. A reader doesn't scan a list by "when was this added" — they scan it by category:
which one's the main site, which are the extras, which are the subfolders wearing their own
hostname. When the visual order doesn't match that mental grouping, every row takes an extra
half-second to place, even though nothing is actually wrong. That friction is real even when
it's minor, and it's exactly the kind of thing that's invisible until someone lives with the
page daily and it never stops needling.

## Same data, sorted on purpose

The fix doesn't touch what's stored or how it's queried — only how the list is presented one
call before it reaches the page. Three groups instead of one: the primary domain first,
every addon domain next, every subdomain last — and *within* each group, the original
insertion order still holds, because that part was never the problem. The domains you added
first within a group still come first. It's only the grouping that changes, layered on top
of an order that was already fine at the small scale.

## The missing label

While sorting this out, a smaller gap surfaced next to it: the list already had a label for
"this is the main domain" and a label for "this is a subdomain of that one" — but a plain
addon domain, sitting between the two, had no label of its own. Nothing wrong, just an
asymmetry once you're looking at all three kinds side by side. Every row now says plainly
what kind of domain it is, not just two out of three.

## What actually finished

The list reads the way it's actually organized, not the way it happened to be typed in.
Nothing about what any given row means changed — only how quickly a glance at the page tells
you which kind of row you're looking at.
