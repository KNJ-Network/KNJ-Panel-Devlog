# Phase 85 - The Update Mechanism Updates Everything, Except the Part That Updates It

Last phase closed out PHP PEAR Packages by widening php-fpm's own sandbox to cover two new
`/usr` subpaths. Before trusting that fix, it was worth actually checking something flagged but
never verified: do the earlier apt-based features (Perl Modules, PHP extensions, OS Package
Manager) share the same exposure? The disposable `knj-test-server` — a real installed copy of the
panel, self-updated the same way a paying customer's server would be — was the obvious place to
find out, since it still had curated Perl modules this server had genuinely never touched.

## The fix that never reached the server it fixed

Upgraded `knj-test-server` from its last real release straight to the one carrying the sandbox
fix. The upgrade reported success. The override file on disk hadn't moved.

The cause was structural, not a typo: `knjpanel-upgrade` — the same script both the real "Update
Now" button and a customer's own terminal both run — resynced exactly one privileged file after
every release, `knjpanel-provision-account`. Every other file this app installs outside its own
directory (the php-fpm systemd override, the queue worker unit, the schedule timer) was simply
never in scope. Every `ReadWritePaths` fix landed across this whole session — vsftpd, ufw,
mariadb.conf.d, postgrey, spamd, awstats, `/etc/knjpanel`, and last phase's `/usr/share/doc` /
`/usr/share/php` — had been silently reaching fresh installs only, never anything already running.

Fixed by teaching the upgrade script to resync every systemd unit this app owns, not just the one
script, restarting (not just reloading) php-fpm specifically when its own sandbox definition
changes — a reload never re-evaluates `ReadWritePaths`, since that's a mount-namespace property
set up once at process start.

The fix itself shipped a real bug on its first live run: one of the three new resync calls
returned non-zero to mean "already in sync," unguarded, under `set -e` — silently aborting every
upgrade where that one file happened to already be current, the common case. Found immediately on
the very next real upgrade, fixed the same way every other bug this session was fixed: read what
actually happened, not what should have happened.

## What the resynced sandbox actually revealed

With the fix genuinely live, the real test: install a Perl module `knj-test-server` had never
seen, through the real UI. `LWP::UserAgent` failed differently than PEAR ever had — not just
`/usr/share/doc`, but `/usr/share/man/man3` for its man pages, and `/usr/bin` itself for
`lwp-download`, an executable the package ships.

That's a different category of problem than the last two. `/usr/share/doc` and `/usr/share/php`
are inert data — opening them widens what dpkg can write, nothing more. `/usr/bin` is on PATH.
Opening it would mean a compromised php-fpm request could write a binary any later process might
run and trust. Widening the sandbox stops being the right answer exactly at that line.

The actual fix was already sitting in this codebase, used for exactly this class of problem:
Package Manager, DB Upgrade, and App Installer all run their real work from the queue worker, not
inline in the web request — because the queue worker's own systemd unit carries no
`ProtectSystem=full` sandbox at all. Perl Modules and PHP PEAR Packages were the two features that
had never made that move, both still running `apt-get install` synchronously inside php-fpm's own
restricted process. Converted both to the same pattern — a run row, a queued job, a live-streamed
log rendered through a new shared terminal partial — and the whole class of "which `/usr` subpath
does this package need this time" question stopped mattering, for these two features, for good.

PHP extension toggling still runs its own `apt-get install` the old way and can hit the same
failure enabling a genuinely new extension. Not fixed this phase — flagged as its own follow-up,
not assumed safe.

## Next

Continuing down the gap list.
