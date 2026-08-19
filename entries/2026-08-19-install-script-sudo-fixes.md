# Phase 100 - Install Script Sudo Fixes, and a Login Bug They Uncovered

The user ran `bootstrap-server.sh` on a genuinely fresh, disposable VM for the first time since
Phase 98's setup wizard shipped — the real test that wizard was built for. It surfaced three
separate, real bugs, found live rather than guessed, each with its own fix.

## Three bugs behind one symptom

All three showed up as some variant of "asks for a sudo password it can never satisfy," but they
were genuinely different causes:

1. **`sudo -u knjpanel` from root still demanded a password.** Proven live, after already being a
   fully-privileged root shell (`sudo su -` first) — nothing missing, nothing racy, and it still
   prompted. `sudo -u` still consults PAM/sudoers even when the caller is root; on this box's
   config, that meant demanding the `knjpanel` account's own password, which has none set to
   anything typeable. Fixed by switching every `sudo -u "$APP_USER"` call to `runuser -u
   "$APP_USER" --`, which requires the caller to already be root but then skips that
   authentication step entirely.
2. **`runuser -l` doesn't work against a nologin shell.** The fix above, done naively, was its own
   regression — `runuser -l "$APP_USER" -c "..."` (login mode) tries to launch the target account's
   shell from `/etc/passwd`, and `knjpanel` deliberately has `/usr/sbin/nologin` (a service account
   shouldn't get an interactive login). Fixed by using non-login mode instead: `runuser -u
   "$APP_USER" -- env HOME="$APP_USER_HOME" bash -c "..."`, with `HOME` derived manually via
   `getent passwd` since non-login mode doesn't set it.
3. **The trial-license request called `sudo dmidecode`, unprivileged, before its own sudoers rule
   existed.** `LicenceService::readHardwareId()` ran `sudo dmidecode -s system-uuid` for the
   hardware UUID — but the install-time trial request runs around step 8/14, and the sudoers rules
   granting `knjpanel` that exact command aren't written until step 12/14. Genuinely zero sudo
   rights at the time this ran, no fix to the calling context could paper over it. Replaced with
   `/etc/machine-id` — systemd-standard, world-readable by design, no privilege needed from any
   caller. Trade-off, not free: unlike the SMBIOS hardware UUID, `/etc/machine-id` regenerates on
   an OS reinstall, which weakens (doesn't remove — see below) resistance to a wipe-and-reinstall
   trial-abuse loop.

Bug 3 explains why my own first verification run looked clean and the user's real run didn't:
mine ran backgrounded (`nohup ... &`), and `sudo` only blocks waiting for a password if it can find
an attached terminal — with none, it fails fast and silently instead. The user's foreground,
terminal-attached run hit the same broken call and actually surfaced it.

## The trial-continuation safety net, checked, not assumed

Read the licence server's own `TrialController` rather than trusting memory of how it was built:
`/trial` looks up any existing trial by `machine_id` and returns that same key/expiry if found,
never issuing a second one. So even with `/etc/machine-id` regenerating on reinstall, the actual
abuse case (repeatedly wiping the box to keep re-triggering a fresh trial) still doesn't work for a
genuine OS reinstall — what's lost is specifically the case of restoring the exact same disk image
from before the panel was ever installed, since that also restores the old `machine-id` file
unchanged and the trial simply continues rather than resetting. Confirmed live: two installs in a
row on a restored-from-snapshot box returned the identical trial key and expiry both times.

## A login bug this uncovered

Working install in hand, the very next login attempt landed on raw JSON (the dashboard's own
polling endpoint) instead of the dashboard page. Root cause, once traced: the header bar's
background stats poll (`server-stats-bar.blade.php`) fetches its own JSON endpoint every 8 seconds
without declaring `Accept: application/json`. If that poll ever fires unauthenticated — a stale
browser tab still open against a box that's since been wiped and reinstalled, for instance —
Laravel's auth middleware treats it as an ordinary page visit: 302 to `/login`, and silently
stashes that JSON URL as the post-login redirect target. The very next real login then lands there
instead of the dashboard. Fixed by declaring the header explicitly (so an unauthenticated hit gets
a clean 401 with nothing stored) and by having the poll stop itself once it sees one, rather than
continuing to hammer a session that's gone.

## Checked the rest of the install/update surface for the same shape

Per standing instruction to check the whole codebase, not just the one instance found:

- `knjpanel-upgrade` (the self-update script behind the Panel Updates page's "Update Now" button)
  had the identical `sudo -u "$APP_USER"` pattern in two places (running migrations, then
  `config:cache`). Switched to the same `runuser` fix. Tested the *old* pattern directly against
  `knj-test-server` first, as a genuine root shell, no missing rules — it succeeded there, meaning
  this specific bug wasn't actually reproducible on that box's config. Kept the fix anyway: same
  call shape that broke elsewhere, so not provably safe everywhere, even where it happened to work.
- DNS-only's own install path (`deploy/dns/setup-dns-only-server.sh`), `DnsOnlyLinkService`, and
  `setup-ftp-server.sh` — clean. No `sudo -u`, no `runuser -l`, no `dmidecode`.
- The half-dozen `Process::run(['sudo', self::SCRIPT, ...])` call sites elsewhere in `app/Services`
  are the opposite direction (the unprivileged `knjpanel` app user elevating itself to root via its
  own scoped NOPASSWD rule) — a different, unaffected mechanism, not the same bug.
- Removed the now-dead `dmidecode` NOPASSWD sudoers rule `bootstrap-server.sh` used to write, since
  nothing calls it anymore.

## Also found, same session: mail/DNS/FTP reading as broken when they're just not installed yet

`bootstrap-server.sh` never installs Postfix/Dovecot/BIND9/vsftpd without a `DOMAIN` — deliberate,
documented, matches how the docs already describe the no-domain install path. But the Service
Status page and dashboard showed all four as plain red "Inactive," indistinguishable from an
actually-broken service. Added a genuine third state — "Not set up yet," with a note pointing at
Server Setup — driven off whether `panel.hostname` is configured, not just whether the systemd unit
happens to be running.

## Verified

Full suite (1,484 tests) and `pint --test` pass. Live: two clean installs in a row on a
snapshot-restored VM, setup wizard through to login working both times, trial licence correctly
continued rather than duplicated. The `knjpanel-upgrade` fix and the login-redirect fix are queued
for their own live upgrade test on `knj-test-server` next, since that box was already outdated and
makes a real, non-synthetic test target.

## Next

Live-verify this release's `knjpanel-upgrade` fix by actually upgrading `knj-test-server`, then
decide whether `DOMAIN` should become mandatory at install time — the original "domain can't point
at a server that doesn't exist yet" reasoning doesn't actually hold, since the server (and its IP)
exists before the panel software is ever installed on it.
