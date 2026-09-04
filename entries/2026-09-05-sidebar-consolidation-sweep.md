# Phase 199 - The Menu That Wouldn't Stop Growing

The Malware Scan and Backups sidebar entries had already been folded from several rows into one
tabbed parent, each time as a side effect of building something else. The prompt to actually go
looking for more of them was a screenshot, pointed at twice for emphasis: "look see backup has 4
menu items." That repetition mattered — it wasn't a one-off complaint about one feature, it was a
request to go check whether the rest of both sidebars had quietly grown the same way.

## What actually collapses safely, and what doesn't

Not every crowded group is the same shape of problem. Some are genuinely one feature wearing
several hats — Migration's two import methods, SSL/TLS's certificates/storage/policy split,
the account panel's MySQL Databases/Browser/Search & Replace trio. Those collapsed the same way
Backups and Malware Scan already had: one sidebar row, an internal tab bar, the drill-down pages
(a backup's browse view, an import's progress screen) left exactly where they were, untouched.

Mail and DNS were a different problem. Fourteen items and nine items respectively, but genuinely
distinct tools rather than facets of one thing — trying to force a three-way split onto either one
risked getting the grouping wrong on infrastructure an admin actually depends on, which is worse
than a long list. Rather than guess, they got a different treatment entirely: a collapsible group
header with an arrow, closed by default so they stop dominating the sidebar, opening automatically
if the page currently on screen happens to live inside one of them, and remembering whatever an
admin chooses via `localStorage` from then on.

## A quieter trap: the sidebar isn't only rendered once

The account-side Databases collapse looked like the same move as the others until a second read of
`AccountFeatureCatalog` made the actual shape of the risk clear: that catalog isn't sidebar-only.
The same list backs the dashboard's feature cards and the admin-side Feature Manager checklist too
— three surfaces reading one array. Simply deleting Database Browser and Search & Replace out of
it to shrink the sidebar would have silently deleted them from Feature Manager and the dashboard as
well, breaking two things nobody was trying to touch. The actual fix was smaller and safer: a
`navHidden` flag that only changes what the sidebar loop skips, leaving the catalog entry itself —
and every other surface reading it — completely intact.

## A one-word collision in the test suite

The full suite came back with exactly one failure after all of this, and it had nothing to do with
menus. `SystemUpdateControllerTest` asserts the Panel Updates page never renders the literal word
"never" — proof that "Last checked" shows a real timestamp instead of its empty-state fallback. A
comment written for the new collapsible-sidebar script happened to use the word "never" in an
unrelated sentence, sitting in a `<script>` tag that renders on every single page, including that
one. Not a real bug — a coincidence between a plain-English code comment and a substring match —
fixed by rewording the comment. Left in here because it's a small, honest reminder that a test
checking raw page text sees every word on the page, not just the ones a change was aiming at.

## What was surveyed and deliberately left alone

Security's remaining dozen items and Services' dozen were both looked at and left as they are —
crowded, but each entry a genuinely different tool, not an obvious grouping waiting to happen. Not
every long list is a problem to solve; some are just long.

Tested (2842/2842), pint clean.
