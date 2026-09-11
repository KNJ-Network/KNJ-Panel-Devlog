# Phase 233 - The Badge That Did Too Much Work

A number in the corner of a dashboard is supposed to be the cheapest thing on the page. It's read
almost by accident, glanced at on the way to somewhere else, never the reason anyone loaded the page
in the first place. Tonight that number turned out to be the reason a login sat spinning for the
better part of a minute — twice, on two different servers, hours apart.

## Caught in the act, not just suspected

The first time it happened, logging into one server produced nothing but a spinning tab. The
obvious suspects were the obvious ones — a network blip, a slow database, something transient that
would clear on its own. Rather than guess, the actual server was checked mid-hang, live, over SSH.
Sitting right there in the process list was a real package-update check, actively running, mid
network fetch, not stuck or deadlocked, just slow. A second incident a few hours later, on a
completely different server reached via a different path, caught the exact same thing happening
again. Twice is a pattern, not a coincidence — and having caught it red-handed both times meant the
fix could be aimed at the real cause instead of a plausible-sounding one.

## A cache that was honest about being slow, in the wrong place

The mechanism itself wasn't broken. A pending-updates count was cached for half an hour, which is a
perfectly reasonable way to avoid asking an external package mirror the same question over and over.
The problem was who got to trigger the *first* answer. Any page that asked this question — a
dashboard loading, a status page refreshing itself in the background, a button that was really just
a "show me the details" click — was equally capable of being the one unlucky visitor who arrived
right after the cache had gone stale. Whoever asked first paid the price of a live check, and if that
mirror happened to be slow that day, whoever asked first was stuck waiting on it, with no way to know
that's what was happening.

## Deciding who's actually allowed to ask

The real fix wasn't a faster check or a smarter cache — it was narrowing who's allowed to trigger one
at all. A page load, a background poll, a details click: none of these are requests to go find out
right now. They're requests to see whatever the last real answer was. Splitting that apart meant
giving every one of those callers a version that can only ever read what's already known, never go
ask fresh — and moving the actual asking to exactly two places that have a real reason to do it: a
new nightly check that runs once, quietly, while nobody's waiting on it, and the explicit button that
says, unambiguously, "check now." Everything else inherits whatever those two left behind, and
nothing else gets to make a visitor wait on an external mirror's mood.

## What actually finished

Loading the dashboard, viewing package details, or watching the status page poll itself can no
longer be the thing that blocks on a live check — proven directly, not just reasoned about: the new
tests hand each of those a process call that would throw if it were ever actually invoked, and none
of them do. The one exception, reaching the same server through a different explicit "check every
server" action, now deliberately still asks for a real answer, because unlike everything else, that
one genuinely means right now. The number in the corner of the dashboard is cheap again.
