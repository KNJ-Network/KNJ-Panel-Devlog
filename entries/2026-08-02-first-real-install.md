# Phase 44 - The First Real Install, on a Server That Had Never Seen This Panel Before

Everything up to this point had been tested by upgrading a box that already had KNJ Panel running on
it. That's a different code path from installing on a server that's never seen any of this, and the
only way to know the install script works is to run it for real on a server provisioned to find out.

So that's what happened: a fresh Ubuntu 24.04 box, and the exact command a real customer would run
— download the install script, run it, nothing else. It found three bugs, each visible only because
this was a genuine first run.

**The first**: the install script located its supporting files relative to itself, which worked
when it sat inside a full checkout, as in every previous test. The real distribution method is a
single standalone file. Fixed by pointing it at the already-installed application directory.

**The second**: re-running the install script — the documented way to recover from a failed first
attempt — failed at the disk quota step, because a previous attempt had already enabled quotas. The
recovery path had never been exercised on a real box. Fixed by skipping the step cleanly if quotas
are already on.

**The third, and the one that mattered most**: no error during the install itself, but logging in
failed afterward, because a directory the application needs at runtime wasn't included in the
release package. It could not have shown up in any test short of "install it and try to log in."
Fixed, and checked by unpacking a real download and inspecting its contents.

With all three fixed, a second full run went cleanly: a real admin login, a trial licence issued
automatically, and a hosting account created and removed through the panel, with everything cleaned
up on removal. Next step is running the fixed version on another fresh server to confirm it needs
no intervention.
