# Phase 237 - A Guard That Was Only Ever Hired Once

Reading through routine logs for anything that mattered turned up two things, both quiet for the same
reason: something built correctly once, with no path to reach the places it was meant to cover.

## Hired, but never actually posted

A defense for hosted sites had been written and tested properly. What hadn't been asked was whether
finishing something for new installs also covers servers that existed before it was written. It doesn't
automatically: a fresh install gets everything decided up to that moment, while an existing server keeps
what it started with unless something explicitly updates it.

## The same shape, twice

A second, unrelated problem had the same root cause: a routine cleanup task was authenticating with a
generic identity that lacked the specific permissions the target required. The task wasn't wrong; it was
a mismatch present since setup, and failing quietly looks the same as failing loudly from a distance.

## Fixing the door, not just the day

The fast fix would have been one manual visit to the affected server. That fixes today but not the next
server or the next similar change. The better fix asks what should check every server automatically, so
the mechanism that catches this case also catches the next.

## What actually finished

When the panel next updates itself, it now also applies protections that older servers were missing, not
only those that existed when it was first written. Both issues are fixed everywhere, not just on the one
server where they were noticed.
