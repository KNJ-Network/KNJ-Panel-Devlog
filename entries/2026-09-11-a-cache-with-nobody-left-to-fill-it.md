# Phase 235 - A Cache With Nobody Left to Fill It

The fix from a few hours earlier did what it was meant to: the dashboard could no longer trigger a live
check of its own. What it hadn't been asked was who does that job instead.

## The fix that solved its own problem too well

Turning a page from "ask live every time" into "read what was last found" is right when the asking was
the problem. But the live version, however clumsy, was also what kept the answer current. The dashboard
now only read a cache, and nothing else had been given the job of filling it. The only other place that
could was a page visited when something already feels wrong. So the cache stayed empty, and the
dashboard honestly reported nothing, which a status light renders like a problem.

## Confirming before chasing

The two services involved were checked directly and were fine. The issue was never whether mail worked,
only whether the dashboard could hear about it.

## Giving the job back, just less often

A small recurring task now does the checking that used to ride along with each visit. The dashboard goes
back to only reading, and something else is responsible for keeping the answer recent.

## A second, smaller room found while the light was on

A status-table column that had been empty now shows the port each service is bound to, read live. That
also explained a long-standing false alarm in the security scan, which was flagging expected ports it
had never been told about.

## What actually finished

A dim status light now means what it says and is refreshed on its own, and the table answers the port
question at a glance.
