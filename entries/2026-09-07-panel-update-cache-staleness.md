# Phase 206 - The Answer That Never Got Asked Again

Three releases were cut back to back and a linked test server was updated to the latest one, but its
row still said an update was waiting. It wasn't a display glitch: the version comparison was running
against an out-of-date number.

## Why the top of the page looked fine anyway

The page's "you're running the latest version" banner reads this server's own version live. What it
compares against is a cached answer to "what is the newest release", kept for an hour so the badge in
the top bar, which appears on every screen, doesn't contact an external server on every click. That
cache predated several of the new releases.

The banner survived the staleness by accident: it only asks whether something newer exists, and a
server already ahead of an old cached answer passes that test. The linked server's row asked a
stricter question, whether its version matches the newest exactly, and a stale answer fails that even
when the real versions agree. Two checks read the same value, one forgiving enough to hide the problem
and one strict enough to expose it.

## Nothing was watching for the moment that mattered

The cache was cleared only after a server finished updating itself. It was never cleared when a
different server learned that a remote server had finished. The fix is at the one place that knows that
moment: the check that confirms a remote update finished now clears the cache as it records success, and
the next reload, which already happens automatically, recomputes it fresh.

This showed up most on the development machine, which is updated differently from a normal install, so
the usual clearing step never ran there. A normal install would rarely sit on a wrong answer for long.
That difference is how it got caught.
