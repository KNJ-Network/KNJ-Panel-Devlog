# Phase 150 - The Log That Was Never Being Written

The unsuppressed chown from last release worked. Read the diagnostics page again a few minutes
later and `/var/lib/roundcube/logs` had genuinely changed owner — `root:roundcube`, a brand new
mtime — the first time in this whole investigation that directory was ever seen to actually move.
Every theory chasing it (`chattr` immutable, a path-scoped AppArmor policy, the `set -e` guard) had
been wrong, or at least not the whole story: chown was never permanently blocked, it just hadn't
been caught in the act before.

So the natural next move was to check the real bug. Loaded `/roundcube/` fresh. Still a 500. Correct
ownership, freshly confirmed, and the actual user-facing failure hadn't budged at all. Which meant
the entire `/logs` chase — four releases of it — was never chasing the root cause. It was chasing an
obstacle in front of the root cause: something that made it *harder to see*, not the thing itself.

Went back to the diagnostics page to check `logs/errors` after that fresh 500. Still didn't exist.
Directory now writable, application error still silent. That ruled out the last remaining version of
the ownership theory too — even a writable log directory wasn't producing a log entry, so whatever
was failing wasn't reaching Roundcube's own logger at all.

That left the other log this action has tailed since the very first version of it:
`/var/log/php8.5-fpm.log`. Reread it end to end, all the way back through this entire session's worth
of repeated 500s. Every single line was `NOTICE`. Reload announcements, start/stop lines. Not one
`WARNING`, not one fatal, across dozens of confirmed failures. That's not a log with nothing to
report — FPM's global log only ever carries the *master process's* own notices. A worker's fatal
goes to its own stderr, and PHP-FPM only captures that into a log if the pool explicitly says so.

Checked the pool config Roundcube actually runs under
(`setup-mail-server.sh`'s `[roundcube]` block in `/etc/php/8.5/fpm/pool.d/roundcube.conf`): no
`error_log` override, no `catch_workers_output`. PHP-FPM's own default for that setting is "no". Every
fatal this pool's workers have ever hit, for this server's entire life, was being thrown away before
it reached any file — not the global log, not Roundcube's own file-log driver (which never got the
chance, if the fatal happened early enough in bootstrap), nowhere. Two separate silences, one shared
cause.

Fixed at both ends: `setup-mail-server.sh`'s pool template now sets `catch_workers_output = yes` and
an explicit `php_admin_value[error_log]` for every future install, and `issue-panel-cert` grew a new
self-repair — unconditional, idempotent, guarded like every other line in this action — that appends
the same two directives to an already-provisioned pool config and reloads PHP-FPM, so a live install
with the old config repairs itself on the next cert issuance without needing SSH. `roundcube-diagnostics`
now tails that dedicated log and prints the live pool config directly, so the next real 500 finally has
somewhere to leave a trace instead of vanishing.

Tested (2506/2506).
