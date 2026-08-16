# Phase 87 - Closing the Loop on the Sandbox Fix

Two phases ago, Perl Modules and PHP PEAR Packages moved off php-fpm's own sandboxed request path
because a genuinely new package could need `/usr/bin` — the one part of `/usr`'s hardening actually
worth keeping, since it's on PATH. That fix flagged one more feature sharing the exact same
exposure without yet sharing the fix: PHP extension toggling, still running its own `apt-get
install` synchronously, inline in the web request, every time an admin enables an extension.

## Same fix, same reasoning

Enabling `php8.3-imagick` on a server that's never had it before makes dpkg write files under
`/usr/share/doc/php8.3-imagick/`, potentially `/usr/share/man/`, and in principle anything else a
given `.deb` happens to ship — the exact same open-ended "which subpath this time" question that
made the earlier fix a move away from the sandboxed path entirely, not another `ReadWritePaths`
entry.

Toggling now follows the identical pattern already proven for Perl Modules and PEAR Packages: a
`PhpExtensionToggleRun` row per attempt, a queued `InstallPhpExtensionJob` doing the real
`apt-get install` from the unsandboxed queue worker, a live-streamed log rendered through the same
shared `partials.install-run-terminal` this session already built. ionCube installation was already
async (a real network download, queued since before this session) — extension toggling was the one
piece of this page still running the old way.

## What's left

Package Manager, DB Upgrade, App Installer, Perl Modules, PHP PEAR Packages, and now PHP extension
toggling all run their real provisioning work from the queue worker. Nothing installs a system
package synchronously inside a php-fpm request anymore.

## Next

Continuing down the gap list.
