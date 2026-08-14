# Phase 82 - The Same Door, Reachable From Both Sides

The admin-side Perl Module Manager (Phase 78) already did the real work: a curated, apt-allowlisted
set of 8 common CGI modules, install state read live off the server, install via the provisioning
script's own validated action. The gap left on the roadmap was purely about who gets to press the
button — a self-service, account-side version of the exact same thing, the way real WHM/cPanel splits
this tool across both its admin and client interfaces.

## Nothing new to secure, because nothing new was built

The account-side controller doesn't reimplement anything — it calls the same `PerlModuleService`,
which calls the same provisioning script action, against the same fixed 8-module list. The only new
code is the permission boundary around it: a `perl_modules` feature gate (so a hosting package can
turn this off), and the same owner-or-permitted-collaborator check every other account-side page
already goes through. An account reaching this page can do exactly what an admin installing a module
on their behalf could already do — nothing more.

## Also: a version number people can actually watch move

While updating the public Latest Updates page for this feature, added a small badge next to the
heading showing the version currently being served to real installs — pulled live from
repo.knj.network's own manifest rather than typed in by hand, so it can never quietly drift from
what a fresh install or a self-update actually lands on. A small thing, but the versioning discipline
this project already holds to (bump on every shipped phase, never automatically cut a release) means
that number is a real, moving signal of how active this thing actually is — worth letting people
see rise.

## Next

Continuing down the gap list.
