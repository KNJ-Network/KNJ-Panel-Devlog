# Phase 46 - Closing Out Service Configuration

Five items had been sitting in the Admin Panel roadmap's Service Configuration section for a
while, three of them not even started: mail relay and filtering, WebDAV access, log rotation,
service certificates, and a service manager. None of them individually looked hard. Taken
together, they turned into one of the more instructive stretches of work in a while, mostly
because three of the five hid a real bug that only showed up once something was actually run
against the live server, not just against a mocked test suite.

Mail filtering turned out simpler than expected in one direction and genuinely dangerous in the
other. Sender-side blocking was just a validation change — Postfix's own address-then-domain
lookup fallback meant one hash map could hold both shapes already. Recipient-side blocking was the
one that needed real care: the settings page that already wrote `smtpd_recipient_restrictions`
rebuilds that entire directive from scratch on every save, which meant a naive recipient-block
feature could silently strip the `reject_unauth_destination` guard the next time anyone touched
Mail Settings or Greylisting, and quietly turn the mail server into an open relay. Wiring the new
block into that same builder instead of setting the directive independently was the whole fix —
and testing it for real against the live Postfix config (not the mocked suite, which passed
cleanly and would never have caught it) turned up a second, smaller bug: the privileged script's
own allowlist of valid restriction phrases hadn't been told about the new one yet, so the very
first live save failed outright. Fixed, redeployed, reverified — `reject_unauth_destination` was
never actually at risk, but the discipline of checking live state directly instead of trusting
green tests is exactly what caught both problems before either one shipped for real.

WebDAV went the way Directory Privacy had already gone — nginx's own `dav_ext` module, bcrypt
htpasswd files written directly by the app, and a shared per-account nginx snippet helper, not a
separate daemon. Genuinely new infrastructure this time was a real Ubuntu package install
(`libnginx-mod-http-dav-ext`) triggered from the admin toggle, gated on `nginx -t` passing before
anything reloads. Verified with an actual account, an actual `curl -u` PROPFIND against the live
server (401 unauthenticated, 207 once credentialed), then torn back down again cleanly.

Log rotation is where the sharpest lesson of the whole batch showed up. The plan called for a
second, panel-owned logrotate file living alongside the stock one nginx already ships, so an nginx
package upgrade could never silently revert an admin's retention settings. Reasonable-sounding, and
wrong: logrotate treats two config stanzas that both glob the same log files as a duplicate entry —
it logs an error and silently skips the second one, while still exiting 0. A second file would have
shipped a setting that looked saved, looked applied, and never actually did anything, and the
mistake would only have been visible by reading `logrotate`'s own debug output line by line, which
is exactly what happened before any of this got written. The real fix was simpler than the original
plan: overwrite the stock file directly. That's no more fragile than the one-line `sed` bump
bootstrap-server.sh already does today for the same file, and it surfaced one more thing hiding
underneath — the privileged script's own sandboxing didn't grant write access to
`/etc/logrotate.d/` at all, so even the corrected version failed on its first live save with a bare
"Read-only file system." Also fixed, also verified live, complete with a real save, a real
`logrotate -d` dry run against the whole combined config, and a real revert back to sane defaults
afterward.

Service certificates ended up being the smallest of the five, deliberately. The panel only ever
issues one certificate, and mail already reuses it — the honest scope here was making that sharing
visible, not building a second issuance path nobody needed. Reissuance now echoes back its own
issued-at, expiry, and issuer, captured with `openssl x509` the same way an FTP account write
already reports back a generated password hash, and the new read-only page just shows what's
already true.

The service manager was the one place a synchronous mistake could have been worse than a rejected
save — this codebase has already been bitten once by a synchronous `systemctl reload php8.5-fpm`
killing the very request that triggered it. Every start/stop/restart here goes through a queued
job, never a direct call from the web request, and stopping nginx, PHP-FPM, MariaDB, or SSH is
refused at three independent points: a disabled button in the UI, the service layer before a job
is ever dispatched, and the privileged script's own allowlist, which doesn't trust the caller's
list either. Restart stays available for all of them, since systemd brings a unit straight back up
from a restart the way it can't from a stop. Proved live, not just tested: a direct attempt to stop
nginx was rejected at both the PHP layer and the shell script directly, and a real restart of
PHP-FPM itself went through the queue, completed successfully, and the request that triggered it
came back completely unaffected — the exact scenario the original bug broke, now provably fixed
rather than just no longer reproducing it by accident.

One more thing needed fixing along the way that had nothing to do with any of the five items: the
dev server's own licence had quietly been a placeholder that had never actually validated,
expiring silently rather than loudly. It's now a real, permanently issued internal licence, run
through the same backoffice code path any paying customer's would go through — capped at
2038-01-01, which turned out not to be a choice but the actual ceiling of the column that stores
it. That's a real constraint worth remembering rather than a deliberate one, and it's now written
down rather than just discovered twice.

Every one of the five items shipped with its own test coverage, its own live verification against
the real running server, and its own commit — 655 tests green by the end, all five marked Live on
the public roadmap, and the whole batch landing as one version bump rather than five, since that's
how the work was actually planned and delivered.
