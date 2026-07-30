# Phase 33 - Disk Quotas That Actually Block Writes, and Bandwidth From Real Logs

Package limits (`disk_quota_mb`, `bandwidth_mb`) have existed on the Package model for a while, but
nothing actually measured or enforced them. Today closed that gap for real, not with an app-level
approximation.

**Disk quota is a real, kernel-enforced ext4 quota.** A write past an account's limit fails with
`EDQUOT` at the filesystem level — the app isn't in the loop at write time at all. That needed
`usrquota`/`grpquota` on the root mount, which turned into a real dig: ext4's modern "quota" feature
can only be toggled while the filesystem is unmounted, which isn't an option for a live server's root
filesystem without a reboot. The classic external-quota-file mechanism doesn't have that restriction
— a live `mount -o remount,usrquota,grpquota /` turned it on immediately, no reboot, persisted in
`/etc/fstab` for next boot and folded into `bootstrap-server.sh` for fresh installs. Verified with an
actual write: writing past a account's quota fails mid-write with "Disk quota exceeded", confirmed via
`repquota`.

**Bandwidth has no OS-level equivalent**, so it's derived by parsing Nginx access logs instead —
deliberately stateless rather than tracking a running total. Every refresh re-sums every log
generation currently on disk for a domain (the live file plus rotated ones, including `.gz`) and
filters by the date embedded in each log line down to the current calendar month. That makes it
self-correcting: a missed refresh cycle, a restart, a manual re-run all produce the same right answer
instead of slowly drifting. The tradeoff is needing a full month of logs available to recompute from,
so Nginx's log retention went from 14 to 32 days.

Disk and bandwidth get enforced differently, on purpose. Disk is already blocked at the kernel level,
so there's nothing left for the app to do there beyond showing the number. Bandwidth has no such
backstop, so an account that goes over its package's allowance gets suspended through the same
`suspend()` path built last session for manual/billing suspensions — one enforcement lever doing
double duty rather than a bespoke one just for this.

Both figures refresh every 15 minutes via a new scheduled command, the first thing in this app to use
Laravel's own scheduler — previously every background job here ran off its own dedicated systemd
timer. Wired up with a new `knjpanel-schedule.timer` running `artisan schedule:run` every minute, the
standard Laravel pattern, so anything scheduled in the future just registers with the same scheduler
instead of needing its own timer unit.

WHM's List Accounts and the account-side page (renamed Disk Usage → Disk & Bandwidth, since it's not
just disk anymore) both show live usage against the package limit with a small progress bar, colored
by how close to the limit it is.

Caught one more thing along the way: four pre-existing test accounts on the dev server predated the
per-domain access-log lines being added to the vhost template a couple of sessions back, so they had
no domain-specific log file for bandwidth tracking to read at all. Confirmed new accounts get it
correctly, patched the four existing ones directly. Verified live end to end on the real dev server —
disk quota applied on account creation and on a package change, a real over-quota write blocked and
confirmed via `repquota`, and bandwidth numbers matching real traffic against the account.
