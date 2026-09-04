# Phase 199 - The Menu That Wouldn't Stop Growing

The Malware Scan and Backups sidebar entries had already been folded from several rows into one tabbed
parent, each as a side effect of building something else. Feedback that Backups still showed four menu
items prompted a check of whether the rest of both sidebars had grown the same way.

## What collapses safely, and what doesn't

Some crowded groups are one feature wearing several hats: Migration's two import methods, SSL/TLS's
certificates, storage and policy split, and the account panel's database tools. Those collapsed into
one sidebar row with an internal tab bar, leaving drill-down pages untouched.

Mail and DNS were different: many genuinely distinct tools rather than facets of one thing. Forcing a
grouping risked getting it wrong on pages an admin depends on, which is worse than a long list. They got
a collapsible group header instead, closed by default, opening automatically when the current page is
inside it, and remembering the admin's choice in the browser.

## A quieter trap: the sidebar isn't only rendered once

The account-side Databases collapse looked like the same move until it became clear that the feature
list behind the sidebar also backs the dashboard cards and the admin Feature Manager. Deleting entries
from it to shrink the sidebar would have silently removed them from those places too. The fix was a
flag that only changes what the sidebar skips, leaving every other surface intact.

## A one-word collision in the test suite

The full suite came back with one failure unrelated to menus. A test asserts that a page never renders
a certain common word, as proof that a timestamp is shown instead of an empty-state fallback. A code
comment for the new script happened to use that word and ships on every page. It was fixed by rewording
the comment, and is a reminder that a test checking raw page text sees every word on the page.

## What was surveyed and left alone

Security's and Services' remaining dozen items each were looked at and left as they are. Not every long
list is a problem to solve.

Tested (2842/2842), pint clean.
