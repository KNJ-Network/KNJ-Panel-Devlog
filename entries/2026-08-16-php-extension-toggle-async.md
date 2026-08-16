# Phase 87 - Closing the Loop on the Package-Install Fix

Two phases ago, Perl Modules and PHP PEAR Packages moved off the web request path and onto the
queue worker, because installing a brand-new package can need to write to locations the web
application is deliberately not allowed to write. That work flagged one more feature in the same
position: PHP extension toggling, which still ran its package install synchronously inside the web
request whenever an admin enabled an extension.

## Same fix, same reasoning

Enabling an extension on a server that has never had it can write to locations that vary by
package, and the earlier fix chose to move the work rather than keep widening what the web
process may write. Toggling now follows the same pattern: a run record per attempt, a queued job
doing the real install, and a live-streamed log shown in the shared terminal view. ionCube
installation was already asynchronous, so extension toggling was the last piece of this page
running the old way.

## What's left

Package Manager, DB Upgrade, App Installer, Perl Modules, PHP PEAR Packages, and now PHP extension
toggling all run their provisioning work from the queue worker. Nothing installs a system package
synchronously inside a web request anymore.
