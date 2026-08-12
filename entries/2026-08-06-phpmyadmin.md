# Phase 67 - Real phpMyAdmin, and Two Bugs Only a Live Server Would Show

A detour from the roadmap's own order — the user wants real phpMyAdmin installed by default and
always visible in the client panel, a permanent fixture rather than something a hosting package
can hide, before we get to the actual Software section item (a proper multi-app installer). Not a
replacement for the existing Database Browser (this app's own custom table/row browser, shipped
earlier this session) — the two sit side by side, a visual browser next to the real, dedicated
tool. The real thing, not a second reimplementation of it.

The interesting design question was single sign-on, not the install itself. phpMyAdmin supports
`auth_type = signon`: point it at a session name and a fallback URL, and if that session already
holds `PMA_single_signon_user`/`password` by the time phpMyAdmin loads, it skips its own login
form entirely. Something has to populate that session first, though, and Laravel and phpMyAdmin
are two entirely separate PHP processes with no shared state by default. The fix follows a pattern
this codebase already uses for a smaller version of the same problem — `DatabaseBrowserService`'s
query runner already hands a real, decrypted `DatabaseUser` password to a privileged action via a
root-owned, `0600`, single-use temp file, never argv, always unlinked after. Extended here: a new
`phpmyadmin-signon-token` action writes the same kind of short-lived (60-second), single-use file,
this time to a directory phpMyAdmin's own dedicated PHP-FPM pool can read via group membership —
and a small bridge script, `knjpanel-signon.php`, deployed outside phpMyAdmin's own package
directory (so `apt upgrade phpmyadmin` can never overwrite it), reads that file exactly once,
populates the session, and hands off. Click "Open phpMyAdmin" in the panel, land inside phpMyAdmin
already logged in as your own MySQL user — never a password typed into a form.

Two real bugs turned up only once this actually ran on a live server, not from writing the code:

The blowfish secret — required by phpMyAdmin for its own internal crypto — is generated the same
way Roundcube's DES key already is elsewhere in this repo: read N raw bytes from `/dev/urandom`,
filter to alphanumeric only, take the first 32. `tr -dc` keeps roughly a quarter of what it sees
(62 of 256 possible byte values), so a single fixed-size read is a bet, not a guarantee. On the
real run here it came up 18 characters short. Fixed by looping until the target length is actually
reached, which is correct regardless of the alphabet's keep-rate — worth a second look at
Roundcube's own copy of this pattern at some point, since it's the identical shape of bug.

The bigger one: the signon bridge script 404'd on every request, immediately, before ever reaching
PHP-FPM. The nginx location pointed `SCRIPT_FILENAME` at the bridge script's real path directly —
correct on its face — but it still included `snippets/fastcgi-php.conf`, which carries its own
`try_files $fastcgi_script_name =404;` line, checking for the file under *this vhost's own
docroot* before the override ever gets a chance to matter. The bridge script doesn't live there on
purpose (see above), so nginx rejected the request on its own before FastCGI was ever involved.
Switched that one location to plain `fastcgi_params`, which carries no such check. Found by
`curl`-ing the exact URL directly against the server and reading nginx's own error log, not by
inspecting the config for the mistake — the config read as correct until traced live.

Live-verified end to end on a disposable account with two real MySQL users (the account's default
one plus a second created through the Quick Setup wizard): the picker correctly listed both, and
launching as the second user landed inside real phpMyAdmin — `User: pmatest_pmatest_user@localhost`
in its own "Database server" panel — showing exactly the one database that user had been granted,
never the other one. MySQL's own grant system is what's actually doing the isolating here, same
principle as the query runner it borrows the credential-handoff pattern from.
