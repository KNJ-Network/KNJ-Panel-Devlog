# Phase 153 - The Quota Plugin That Needed a Friend

Testing webmail on a single-server install turned up a second, unrelated bug. Roundcube logged in fine,
then showed random-looking "Internal error occurred" banners.

The mail server's own log had the real story on every login: a mail-server plugin could not start because
a plugin it depends on was not also enabled for that kind of session. Mail delivery had it; mail reading
did not. It surfaced now because this was the first end-to-end webmail login test on a plain single-server
install rather than the linked Mail Only setup used for most testing.

The fix is a one-line addition to the generated configuration, and it is applied the next time a server's
hostname or certificate is confirmed, so installs that already exist pick it up with no separate migration.

A second, much smaller bug found the same day: the Roundcube diagnostics page was reading a log file under
the wrong name. The name was corrected.

Tested (2506/2506).
