# Phase 85 - The Update Mechanism Updates Everything, Except the Part That Updates It

The previous phase closed out PHP PEAR Packages by widening the web application's hardening
allowances for two new locations. Before trusting that fix, we checked something flagged but never
verified: whether the earlier package-based features shared the same exposure. A disposable test
server, a real installed copy of the panel updated the way a customer's server would be, was the
obvious place to find out.

## The fix that never reached the server it fixed

The test server was upgraded straight to the release carrying the fix. The upgrade reported
success, but the fix hadn't taken effect.

The cause was structural: the upgrade script only resynced one privileged file after each release.
Other files the app installs outside its own directory were never in scope, so earlier hardening
adjustments had been reaching fresh installs but not servers being updated. The script now resyncs
every service definition the app owns, and restarts the web application's service when its
definition changes, since a reload doesn't re-apply those settings.

The fix shipped with one bug of its own on its first live run: a resync step returned a non-zero
status to mean "already in sync", which aborted upgrades where that file was already current. It
was found on the next real upgrade and fixed.

## What the resynced configuration revealed

With the fix live, installing a Perl module the test server had never seen failed in a different
way than PEAR had, this time touching a location that other processes trust and run from. Opening
that location to the web application would be the wrong trade. Widening hardening allowances stops
being the right answer exactly at that line.

The better fix was already in the codebase: Package Manager, DB Upgrade, and App Installer run
their real work from the queue worker rather than inline in the web request. Perl Modules and PHP
PEAR Packages were converted to the same pattern: a run record, a queued job, and a live-streamed
log. That made the whole class of "which location does this package need this time" questions moot
for those two features.

PHP extension toggling still ran inline at this point and was handled in the following phase.
