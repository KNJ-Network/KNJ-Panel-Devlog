# Phase 82 - The Same Door, Reachable From Both Sides

The admin-side Perl Module Manager (Phase 78) already did the real work: a curated set of 8 common
CGI modules, install state read live off the server, and installation through the provisioning
script's validated action. The remaining gap on the roadmap was purely about who gets to press the
button: a self-service, account-side version, the way real WHM/cPanel offers this tool in both its
admin and client interfaces.

## Nothing new to secure, because nothing new was built

The account-side controller doesn't reimplement anything. It calls the same module service and the
same fixed 8-module list. The only new code is the permission boundary around it: a feature gate,
so a hosting package can turn it off, and the same owner-or-permitted-collaborator check every
other account-side page goes through. An account reaching this page can do exactly what an admin
installing a module on their behalf could already do, nothing more.

## Also: a version badge

While updating the public Latest Updates page for this feature, we added a small badge next to the
heading showing the version currently being served to installs, pulled live from the release
manifest rather than typed by hand, so it can't drift from what a fresh install or self-update
actually receives.
