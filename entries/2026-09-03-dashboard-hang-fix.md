# Phase 180 - The Page That Was Never Going to Load

A real production report: the admin dashboard on a satellite server just sat there. Not an error, not
a 500 — a spinner, forever, on a page that had loaded fine an hour earlier. My own browser tool
reproduced the exact same thing, from a different machine, with no session cookies at all. Whatever
this was, it wasn't the browser, and it wasn't the network.

## A page that half-loads is a page mid-request

The page's own title and header text rendered instantly. The spinner underneath never went away.
That split is the whole diagnosis in miniature: the HTML shell arrived fine, and something the page
asks for *after* that never came back. Chasing it meant watching the actual network requests the
page made, not staring at the URL bar — and one request stood out immediately: a JSON endpoint the
dashboard polls for live service status, sitting there with a raw connection timeout, while every
other request on the same page succeeded in under a quarter of a second.

## A call with no ceiling

That endpoint's whole job is asking systemd, once per service unit, whether the unit is active —
`systemctl is-active nginx`, and so on down a short list. A completely ordinary, in-memory,
should-be-instant systemd query. The code calling it had no explicit timeout at all, which meant it
inherited Symfony's own default: sixty seconds. Fine for one call. Not fine for a loop over nine of
them, if even one is genuinely stuck — and worse, that one stuck call wasn't caught locally. It
propagated straight out of the whole method, discarding every OTHER unit's result along with it —
eight already-successful checks, thrown away because the ninth took too long. On a real box with
enough units to check, that's a worst case measured in minutes, not seconds, before the page finally
gave up and showed nothing at all.

## Fixing the shape, not just the number

The obvious fix — add a timeout — isn't the whole fix on its own. A five-second timeout still throws
away every other unit's answer if it's not caught in the same place it's thrown. The real fix moves
the catch inside the loop, not around it: each unit gets its own short timeout and its own recovery,
so a stuck one reports "inactive" for just that row while every other row still shows its real,
correct status. The difference isn't just speed — it's that a partial failure now looks like a
partial failure, one red dot in a page that otherwise works, instead of an outage.

## Verifying it against the shape of the actual bug

A regression test reproduces the real failure mode directly: one unit configured to hang while the
rest resolve normally, asserting every other unit's status still comes back untouched — not just that
the page doesn't crash, which the previous test already technically confirmed and still missed the
real problem. Live-verified on the actual server class this hit: all nine applicable service units
now report correctly, cleanly, with the fix in place.

Tested (2709/2709).
