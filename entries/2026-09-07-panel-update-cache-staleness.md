# Phase 206 - The Answer That Never Got Asked Again

Three releases cut back to back, a linked test server updated to the latest one — and its own row
still insisted an update was waiting. Not a display glitch. The version comparison was quietly
running against the wrong number.

## Why the top of the page looked fine anyway

The page's own "you're running the latest version" banner never lied — it read this box's own
version straight off disk, live, every time. What it compared that number against, though, was a
cached answer to "what does the update server currently call the newest release" — kept around for
an hour at a time so a page every admin passes through constantly (this same check backs a small
badge in the bar at the top of literally every screen in the panel) doesn't hit an external server
on every click. That cache had been sitting there since the very first check of the day, long before
several new releases existed, and nothing had ever told it to look again.

The banner's own comparison happened to survive that staleness by accident: it only ever asks "is
there something newer than me out there," and a box that's already raced ahead of a months-old
cached answer trivially passes that test regardless of how wrong the cached number actually is. The
linked server's row asked a stricter question — "does your version match the one I think is
newest, exactly" — and a stale answer fails that test even when the real, current versions on both
sides agree completely. Two panels reading the same underlying value, one forgiving enough to hide
the problem, one strict enough to expose it.

## Nothing was watching for the moment that mattered

There's already exactly one place this cache gets cleared: right after a server finishes updating
itself, so its own next look at the page reflects reality instead of an hour-old guess. That covers
a server updating on its own — it never covers a *different* server, sitting elsewhere, learning
that some other server just finished updating. The whole point of triggering an update on a linked
server from here is that nobody has to go check on it individually — but the one number that
decides whether its little badge reads correctly was never told to refresh at the exact moment that
badge stopped being accurate.

Fixed at the only place that actually knows the real moment it matters: the same background check
already responsible for confirming a remote update finished now clears that cached answer itself,
the instant it records a genuine success. The very next reload — which already happens automatically
the moment that update wraps up — recomputes it fresh, no separate click required.

Worth being honest about scale here too: this bit hardest on the box running this whole build,
since it updates itself by pulling from git directly rather than through the same self-update path
every real install uses — meaning the one thing that already clears this cache correctly never ran
here at all, letting the staleness go effectively unlimited instead of capped at an hour. A real
install would rarely sit on a wrong answer for long. This one could have sat on it indefinitely,
which is exactly how it got caught.
