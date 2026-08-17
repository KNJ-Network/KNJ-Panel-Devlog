# Phase 97 - Return to Admin, and Making 0.15.99 Real

Two small closes to end the night: a live-reported impersonation bug, and finally cutting a
release so tonight's work actually shows up as one.

## The 404 with no way back

Testing the "return to admin" flow after impersonating an account turned up a dead end: click it
twice, or land on a stale bfcache-restored page after the first click already went through, and
`ImpersonationController::stop()` threw a hard 404. The account owner was still validly logged in
either way — there just wasn't an `impersonator_id` left in the session to hand control back to,
since `session()->pull()` is one-shot by design. Aborting with a 404 in that state strands someone
who's already authenticated, on an error page, with no link back to anywhere.

The fix is a graceful redirect to the account's own dashboard instead of an abort — if there's no
impersonator on record, the safest assumption is that control was already handed back, so just
land them somewhere real. The more interesting part was the test suite: the old test asserted the
404 as the *correct* behavior (`assertNotFound()`), so this wasn't a case of a gap in coverage —
the test had actively encoded the bug as intentional. Fixed the controller, rewrote that test to
assert the redirect, and added a second test covering the double-submit case directly.

## Cutting 0.15.99

Separately: `VERSION` had been bumped every phase tonight, per the usual "bump every phase, cut
only when asked" split, but nothing had actually been cut to `KNJ-Panel-Builds` since 0.15.95
loaded three phases earlier. The Panel Updates page's "Recent releases" panel was — correctly —
showing 0.15.95 as the latest real release, which read as broken from the outside even though it
was just accurately reporting that nothing newer had been published yet.

Ran `cut-release.sh` against a clean, pushed HEAD to cut v0.15.99 for real: User Manager, the
Exim/Mail Settings widening, and this impersonation fix, rolled into one changelog. Pushed to
`KNJ-Panel-Builds`, picked up by `knjpanelbuilds-sync.timer` within the minute, and confirmed live
at `repo.knj.network/manifest.json` before calling it done — Panel Updates on panel-dev now shows
0.15.99 as current with 0.15.95 correctly rolled into history underneath it.

## Next

DNS-only server profile work starts next session — bigger scope, deliberately not started tonight.
