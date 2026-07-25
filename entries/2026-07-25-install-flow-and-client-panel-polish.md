# Phase 15 - Rethinking the Install Flow, Client-Panel Polish, and a Second Audit

## The install script's own blind spot

The bootstrap script from last time had one real design flaw: it tried to issue a real TLS
certificate during install, which only works if the domain already points at the server before
installation starts. Fine for our own test servers, unrealistic for a normal install.

The fix follows the common shape for install scripts: install with a self-signed certificate,
print the access details and a freshly generated one-time admin login at the end, and defer the
real domain and certificate to a first-run "Server Setup" step inside the panel. Log in, set the
real domain, and the panel issues its own certificate and switches over.

Testing this end to end surfaced one real bug: creating the very first hosting account failed
because a fresh install had no server record registered for itself. Fixed by having the install
register it as part of setup.

## Every background action now says what happened

Pages that kick off a background task (installing a PHP version, issuing a certificate, toggling
a PHP extension) used to give no indication whether it worked, failed, or was still going. Every
one of those pages now shows a spinner while the task runs and polls until it resolves to a clear
success or failure, including the actual error message on failure.

## Closing gaps in real account creation

With the panel usable end to end, the next pass compared account creation against how established
hosting panels behave, and closed a few gaps:

- **Usernames are now generated from the domain**, not manually typed.
- **Passwords are generated automatically** and shown once, immediately after account creation.
- **"Log in as user"** — a button on every row of the accounts list that lets an admin open that
  customer's own panel, with a clear banner and a one-click way back to the admin view.
- The accounts list gained username, contact email, server IP, and creation date columns.
- The two login pages (admin and customer) now carry distinct badges and accent colors.

## A second security audit

With a second server bootstrapped and this round of changes shipped, it was time for another full
pass over permissions, database grants, session configuration, and every privileged action added
since the last audit. It also re-checked the previous audit's biggest finding — the web app running
under a personal login — which the new dedicated service account was built to fix.

Everything came back clean apart from one cosmetic permission on a directory.

## What's next

Account-side features: multi-domain hosting per account, and giving account owners visibility into
their own traffic and logs.
