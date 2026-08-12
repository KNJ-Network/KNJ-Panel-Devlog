# Phase 52 - The Databases Section, Closed Out

Two items left on the Databases roadmap — a visual database browser and a set of maintenance
tools. Both shipped this session, alongside two real bugs found the only way that actually proves
anything: by clicking through the finished feature against the real server rather than trusting a
green test suite.

The database browser deliberately goes further than "manage what the panel created." It lists
every database on the box — the panel's own `knj_panel` included — because that's what a real
database administration tool gives you, and pretending otherwise would just mean reaching for a
terminal the moment something outside the panel's own accounts needed a look. Browsing tables and
rows is the easy, safe half. The query runner is the deliberate one: it runs whatever SQL an admin
submits, completely unrestricted by statement type — SELECT, INSERT, UPDATE, DELETE, DDL, all
allowed. That sounds like a bigger risk than it is: this role already has other unrestricted
root-equivalent actions (Server Setup, a System reboot, Panel Update's self-upgrade), so a query
runner doesn't introduce a new trust tier, it just extends the existing one to the one remaining
place an admin would expect it.

Maintenance Tools adds a live server status view, the real MySQL process list with a per-process
Kill button, a slow query log toggle, and one-click repair/optimize for any database — mysqlcheck
under the hood, which turns out to be a no-op for InnoDB's own crash-recovery repair but genuinely
useful for the optimize pass regardless of engine.

Two bugs turned up during live verification, both worth remembering:

**mysql's batch-mode escaping, shown raw instead of reversed.** The row browser splits mysql's
tab-separated output on literal tabs — which only works because mysql's `-B` batch mode escapes any
real tab/newline/backslash *inside* a field's own value first. Nothing was reversing that escaping
before display, so a log column with embedded newlines showed up as literal `\n` text in the UI
instead of real line breaks. Fixed with a single-pass `strtr()` unescape — deliberately not
sequential `str_replace()` calls, since replacing `\\` before `\n` (or vice versa) risks corrupting
an already-legitimate escape sequence depending on order.

**A hardened systemd unit blocking its own feature's write.** The slow query log toggle's
`SET GLOBAL` half worked immediately, but the config file it writes for persistence-across-restart
failed with a bare "Read-only file system" — `/etc/mysql/mariadb.conf.d` wasn't in php-fpm's
`ReadWritePaths` allowlist. This is the same class of issue that's hit this codebase before
(`/etc/vsftpd.conf`, `/etc/logrotate.d`, `/etc/ufw` each needed the same fix when their own features
shipped) — `ProtectSystem=full` makes `/etc` read-only inside php-fpm's mount namespace even for a
process escalated to root via sudo, since sudo changes the effective user but never escapes a mount
namespace. One more line added to the allowlist, and — because that allowlist lives in the app repo
under `deploy/systemd/` and auto-deploy already diffs and resyncs it on every push, restarting
php-fpm itself when a drop-in changes — the fix ships the same way every other config file in this
app does: committed, pushed, self-applied.

One more thing surfaced while chasing the second bug, not new this session but worth closing the
loop on: the real `licence.valid` setting on `panel-dev` had been sitting corrupted since a test run
apparently touched the live database, before this session's `tests/bootstrap.php` fix (which
deletes any cached Laravel config before the `Application` object even exists) actually had a
chance to take hold. On top of that, `Setting::get()` caches forever via `Cache::rememberForever`,
so a raw database write during the repair silently didn't apply until the setting cache was
explicitly cleared too — a second footgun layered on the first. Cleared the corrupted rows, cleared
the cache, and requested a genuinely fresh trial licence from the real licence server rather than
hand-writing values back in — same principle as every other "regenerate from source of truth"
pattern in this codebase.
