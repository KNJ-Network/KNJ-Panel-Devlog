# Phase 67 - Real phpMyAdmin, and Two Bugs Only a Live Server Would Show

A detour from the roadmap's order: real phpMyAdmin installed by default and always visible in the
client panel, ahead of the Software section's multi-app installer. It doesn't replace the
existing Database Browser (the panel's own table and row browser); the two sit side by side, a
visual browser next to the real, dedicated tool.

The interesting design question was single sign-on, not the install itself. The panel and
phpMyAdmin are separate applications, so the panel hands over the account's own database
credentials through a short-lived, single-use mechanism rather than ever putting a password on a
command line or in a form. Click "Open phpMyAdmin" in the panel and you land inside phpMyAdmin
already logged in as your own database user.

Two bugs turned up only once this ran on a live server. A generated secret came out shorter than
intended because of how random bytes were filtered, and was fixed by generating until the target
length is reached. And a routing configuration issue made the sign-on step return 404, found by
requesting the URL directly and reading the web server's own log rather than inspecting the
config, which read as correct.

It was verified end to end on a disposable account with two database users: the picker listed
both, and launching as the second landed inside phpMyAdmin as that user, showing exactly the one
database that user had been granted. The database's own permission system does the isolating.
