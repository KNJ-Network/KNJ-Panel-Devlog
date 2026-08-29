# Phase 151 - Array First, Finally Last

The dedicated pool log from the last release came back empty. Not a good sign on its own — but the
global `php8.5-fpm.log`, tailed since the very first version of this whole investigation, had
something new in it for the first time: not another NOTICE, a real `WARNING` line, timestamped right
after this release's own reload, carrying the actual stderr of a worker that had just died.

`catch_workers_output` doesn't only feed the pool's own configured `error_log` — when FPM catches a
worker's stderr, it also logs it through the master process's own log, prefixed `[pool roundcube]
child NNNN said into stderr:`. That's where it landed. Whatever kept the per-pool file from filling
(most likely just the reload's own timing against exactly when the next request happened to land) turned
out not to matter, because the global log caught it anyway.

The message itself, five lines of stack trace attached:

    PHP Fatal error:  Cannot redeclare function array_first() in
    /usr/share/roundcube/program/lib/Roundcube/bootstrap.php on line 308

Every request to `/roundcube/` has been fataling here, every single time, for this server's entire
life. `array_first()` and `array_last()` are new native functions in PHP 8.5. Roundcube's own
bootstrap.php — written for older PHP versions that don't have them — defines its own versions of both
as a compatibility shim. If that shim isn't guarded with `function_exists()`, loading it on a PHP
version that already has the real thing is an immediate fatal, before Roundcube gets anywhere near
its own request handling, its own config, or its own error handler. That's the second silence this
investigation already confirmed directly: `logs/errors` never got written to even with a writable
directory, because Roundcube's own logger never got the chance to initialize.

Every theory this investigation built and discarded — `chattr` immutable, a path-scoped AppArmor
policy, `set -e` aborting a self-repair, a chown that only sometimes took — was chasing a real,
separately-confirmed bug in the panel's own provisioning script. None of them were ever going to
explain the actual 500, because the actual 500 has nothing to do with file permissions at all. It's a
function name collision between this Roundcube build and this PHP version, sitting in a file this
codebase has never touched.

Added a read-only diagnostic before writing the fix: PHP's own `function_exists()` view of both names,
the installed `roundcube-core` package version, and the exact polyfill source with line numbers —
confirming the precise shape rather than assuming it from the fatal's message alone, same discipline as
every step in this chase. Whatever that confirms, the fix itself is next: guard the collision, or
route around it, in whichever way keeps working across a future `roundcube-core` package update instead
of just papering over today's version.

Tested (2506/2506).
