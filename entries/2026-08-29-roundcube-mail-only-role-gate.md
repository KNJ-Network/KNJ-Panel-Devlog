# Phase 141 - Roundcube Was Installed on Mail Only, Just Never Wired Up

Found while working through a pre-production readiness audit on a real four-server stack (dedicated
main, mail, and two DNS boxes): clicking "Open Webmail" on an account whose mail runs on the linked
Mail Only satellite hit a plain 403.

The install had done everything right — the active mail server was correctly switched, Roundcube was
installed on that satellite, credentials were correct. The gap was narrower than that: the nginx
vhost's shared Roundcube location block was written only for the `main` role, on the assumption that
a Mail Only or DNS Only box never has anything at `/roundcube` to route to. True for DNS Only, wrong
for Mail Only — the mail-setup script installs and symlinks Roundcube there too, since that's exactly
where mail (and therefore webmail) actually runs once a satellite is switched on. The app was fully
installed with no nginx location ever pointing at it.

Chased a red herring first: re-issuing the server's hostname/cert repeatedly on the theory the live
vhost had drifted from a fixed template. It hadn't — the vhost was regenerating correctly on every
call, just correctly reproducing the same role gate each time. The actual bug was one line back, in
the role check itself, present in both `bootstrap-server.sh` (first install) and the live
`issue-panel-cert` provisioning action (every later hostname/cert change) — both now gate Roundcube to
"not DNS Only" instead of "only Main," while phpMyAdmin and the client account area correctly stay
main-only, since neither of those actually installs anywhere else.

Also added real logging to KNJ Webmail's own folder-listing failure path, found in the same pass — a
different symptom (webmail loads and authenticates fine, then can't list folders) that a broad
exception catch had been quietly swallowing into a generic banner with no trace of the real IMAP
error anywhere. Couldn't run the underlying cause fully to ground this round (the dev server's own
test satellite was offline mid-investigation), but it's no longer invisible next time it happens.

Tested (2506/2506).
