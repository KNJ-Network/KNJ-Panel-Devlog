# Phase 15 - Rethinking the Install Flow, Client-Panel Polish, and a Second Audit

## The install script's own blind spot

The bootstrap script from last time had one real design flaw: it tried to issue a real TLS
certificate during install, which only works if the domain is already pointed at the server
*before installation even starts*. Fine for our own test servers where that's true by
construction, but unrealistic for a normal install — nobody points DNS at a server that doesn't
exist yet.

The fix follows the same shape every mature install script uses (Proxmox, pfSense, and plenty of
others do a version of this): install with a self-signed certificate on the bare IP, print the
IP, ports, and a freshly generated one-time admin login at the end, and defer the real domain and
certificate to a first-run "Server Setup" step inside the panel itself. Log in with the printed
credentials, set the real domain, and the panel issues its own certificate and switches over.

Testing this end to end surfaced one real bug: creating the very first hosting account failed
with a 404. The cause was almost embarrassingly simple once found — a fresh install has no
"server" record registered for itself yet, and account creation looks one up unconditionally.
Fixed by having the install script register the server as part of setup, the same way it now
handles the admin account.

## Every background action now says what happened

A running theme once real testing started: pages that kick off a background task (installing a
PHP version, issuing a certificate, toggling a PHP extension) would just... sit there. No
indication whether it worked, failed, or was still going, unless you knew to manually refresh.
Every one of those pages now shows a spinner while the task runs and polls automatically until it
resolves to a clear success or failure state — including the actual error message on failure, not
just a red X. Small change, but it's the difference between a panel you can trust and one you
have to double-check by re-running the same action to see if it silently worked the first time.

## Closing gaps in real account creation

With the panel now actually usable end to end, the next pass was checking account creation
against how established hosting panels really behave, and closing a few gaps:

- **Usernames are now generated from the domain**, not manually typed — a short identifier
  derived from the domain being registered, rather than whatever an operator happens to type into
  a name field.
- **Passwords are generated automatically** and shown exactly once, immediately after account
  creation — never stored anywhere retrievable, never shown again after leaving the page.
- **"Log in as user"** — a button on every row of the accounts list that drops an admin straight
  into that customer's own panel with no password prompt, with a clear "you're logged in as X"
  banner and a one-click way back to the admin session.
- The accounts list itself gained columns it was missing — username, contact email, server IP,
  and creation date.
- The two login pages (admin and customer) were previously pixel-identical, which made it easy to
  lose track of which one you were looking at. They now carry distinct badges and accent colors.

## A second full security audit

With a second server now fully bootstrapped and this round of changes shipped, it was time for
another full pass — sudoers, file permissions, database grants, listening ports, session cookie
configuration, and a fresh line-by-line read of every privileged script action added since the
last audit (particularly the newest ones: certificate issuance, PHP extension toggling, ionCube
installation). It also specifically re-checked the previous audit's biggest finding — the web app
running under an over-privileged personal login — since that's exactly what this server's new
dedicated service account was built to fix.

Everything came back clean, bar one purely cosmetic permission on a directory that isn't
reachable by anything else on the box anyway. Full detail, as always, stays in the private
engineering log.

## What's next

Server-level features are in good shape now. The next gaps worth closing are on the account
owner's side: multi-domain hosting per account, and giving account owners some real visibility
into their own traffic and logs — both flagged during the earlier demo walkthrough as things a
real hosting panel has that this one doesn't yet.
