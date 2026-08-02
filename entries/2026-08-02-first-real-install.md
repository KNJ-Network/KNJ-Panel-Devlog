# Phase 43 - The First Real Install, on a Server That Had Never Seen This Panel Before

Everything up to this point had been tested by upgrading a box that already had KNJ Panel running
on it. That's a genuinely different code path from installing on a server that's never seen any of
this before — and the only way to actually know the install script works is to run it, for real, on
a server provisioned specifically to find out.

So that's what happened: a fresh Ubuntu 24.04 box, nothing on it, the exact command a real customer
would run — download the install script, run it, nothing else. It found three real bugs, each one
only visible because this was a genuine first run rather than a re-run of something already proven.

**The first**: the install script computes its own working directory to find its supporting files
(systemd units, the landing page, the mail/DNS setup scripts). That worked fine when the script
still lived inside a full checkout of the whole project — which is how every previous test had run
it. But the actual, real distribution method is a single standalone file, downloaded on its own,
sitting alone with no supporting files anywhere nearby. First real run, first real failure, at the
exact step that needed one of those files. Fixed by pointing it at the already-installed application
directory instead, which has everything it needs by the time that step runs, regardless of how the
script itself got there.

**The second**: re-running the install script — the documented way to recover from a failed first
attempt — hit a wall on the exact thing the first bug caused. Disk quota setup checks the quota
files for consistency as part of turning them on, and that check refuses outright once quotas are
already active, which they were, because a previous attempt had already gotten that far. The
"recover from a failure" path had genuinely never been exercised for real before, on a real box.
Fixed by skipping the whole step cleanly if quotas are already on.

**The third, and the one that mattered most**: no error at all during the install itself — every
step reported success, right through creating the admin account and licence. It only showed up
once the obvious next thing happened: trying to actually use the panel. A directory the application
needs for logging and session storage was never being included in what gets shipped, at all, for
any install, ever — an omission in exactly how many files should be packaged, not a bug that would
show up in any test short of "install it and try to log in." Everything up to that point works
without it; logging in doesn't. Fixed by including it, and — because this is the kind of thing that
should never be silently missing again — checked directly by unpacking a real download and counting
what's actually inside it.

With all three fixed, a second full run went cleanly start to finish: real admin login, a real trial
licence issued automatically and confirmed matching on both sides, and a real hosting account
created and removed through the panel itself — actual system user, actual site, actual database,
all present, all cleaned up correctly on removal. Next: run the fixed version, start to finish, on
another fresh server, and see if it needs zero intervention this time.
