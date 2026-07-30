# Phase 33 - Disk Quotas That Actually Block Writes, and Bandwidth From Real Logs

Package limits for disk and bandwidth have existed for a while, but nothing measured or enforced
them. Today closed that gap for real, not with an app-level approximation.

**Disk quota is a real, kernel-enforced filesystem quota.** A write past an account's limit fails
at the filesystem level — the app isn't in the loop at write time. Enabling it on a live server
without a reboot needed a different mechanism than the filesystem's newer built-in one, which can't
be toggled while mounted. It was folded into the install script for fresh installs. Verified with
an actual write past an account's quota, which failed mid-write with "Disk quota exceeded".

**Bandwidth has no OS-level equivalent**, so it's derived from web server access logs,
deliberately stateless rather than tracking a running total. Each refresh recomputes the current
month from the logs, which makes it self-correcting: a missed refresh or a restart produces the
same right answer. The tradeoff is needing a month of logs, so log retention was extended
accordingly.

Disk and bandwidth are enforced differently, on purpose. Disk is blocked at the kernel level, so the
app only shows the number. Bandwidth has no such backstop, so an account that goes over its
package's allowance gets suspended through the same suspend path used for manual suspensions.

Both figures refresh on a regular schedule using the app's own scheduler. The account-side page
was renamed Disk Usage to Disk & Bandwidth, and List Accounts and that page both show live usage
against the package limit with a progress bar.

One more thing caught along the way: older test accounts on the development server predated the
per-domain log files and had none for bandwidth tracking to read. New accounts get them correctly;
the existing test accounts were patched.

Verified live on a development server: quota applied on account creation and on package change, an
over-quota write blocked, and bandwidth numbers matching real traffic.
