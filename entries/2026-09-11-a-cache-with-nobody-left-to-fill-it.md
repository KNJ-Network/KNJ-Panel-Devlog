# Phase 235 - A Cache With Nobody Left to Fill It

The fix from a few hours earlier had done exactly what it was supposed to do. The dashboard could no
longer trigger a live check of its own. What it hadn't been asked, in the moment, was a quieter
question: once the dashboard stops doing that job, who does it instead?

## The fix that solved its own problem too well

Turning a page from "ask live, every time" into "read whatever was last found" is the right move
when the asking was the problem. But an "ask live" version, however clumsy, has one property worth
noticing on the way out: it's also the only thing keeping the answer current. Remove it cleanly
enough, and you can remove that too, without meaning to.

That's what happened here. A dashboard that used to check-and-cache became a dashboard that only
reads a cache — and nothing else in the building had ever been given the job of writing to it. The
one other place that could was a page nobody visits on a schedule, only when something already feels
wrong. So the cache sat there, correctly empty, forever, and the dashboard read that emptiness
honestly: not "broken," just "nothing here to report" — which, rendered as a status light, looks
exactly like a problem even when there genuinely isn't one.

## Confirming the ghost before chasing it

The instinct to double check rather than assume paid off immediately: the two services in question
were, in fact, both completely fine, reachable directly and confirmed running. The bug was never in
whether mail worked. It was in whether the dashboard could ever hear about it.

## Giving the job back, just less often

The earlier fix removed a page load's ability to trigger a check. It didn't need to remove checking
altogether — it needed to move that responsibility somewhere that isn't a person's own click. A small
recurring task, running on its own schedule, now does the asking that used to ride along on every
visitor. The dashboard goes back to only ever reading, the way it should, but now there's someone
else whose entire job is making sure there's something recent to read.

## A second, smaller room found while the light was already on

Looking at the same status table while all this was fresh turned up an easy addition: a column that
had been sitting empty, now showing exactly which port each service is really bound to — not a
guess, not a static list, but the real, current answer, pulled the same way the security scan's own
exposure check pulls it. That, in turn, explained something else that had been quietly unfair for a
while: a scan that could never reach a clean result, flagging three ports that were never a mystery
at all, just never told they were expected.

## What actually finished

A status light that goes dim now means what it says, refreshed on its own, by something whose only
task is refreshing it — not by whoever happens to be looking. And a table that used to leave a
question unanswered now answers it directly, in the same glance.
