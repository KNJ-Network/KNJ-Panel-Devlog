# Phase 97 - Return to Admin, and Making 0.15.99 Real

Two small closes to end the day: an impersonation bug, and cutting a release so the day's work
shows up as one.

## The 404 with no way back

Testing the "return to admin" flow after impersonating an account turned up a dead end: clicking it
twice, or landing on a stale cached page after the first click already succeeded, produced a hard
404. The account owner was still validly logged in, but there was no impersonation record left to
hand control back to, since it is consumed on first use by design. Aborting there strands someone on
an error page with no link back.

The fix is a graceful redirect to the account's own dashboard. The old test had asserted the 404 as
correct behavior, so it was rewritten to assert the redirect, and a second test covers the
double-submit case.

## Cutting 0.15.99

The version number is bumped every phase and a release is cut only when asked, so nothing had been
published since 0.15.95. The Panel Updates page was accurately showing 0.15.95 as the latest
release, which read as broken from the outside. Release 0.15.99 was cut for real: User Manager, the
mail settings widening, and this fix, in one changelog. It was confirmed live on the release
manifest, and Panel Updates on the development server now shows 0.15.99 as current with 0.15.95
rolled into history.
