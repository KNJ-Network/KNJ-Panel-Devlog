# Phase 21 - Choosing Your Own Password on Account Creation

A small but genuinely useful gap: Create Account has always auto-generated the owner's password
and shown it once, which is the right default and matches real cPanel/WHM — but there was no way
to set one explicitly instead, which matters when handing an account straight to a real client who
wants to pick their own from the start rather than being given a random string to change later.

## What shipped

A toggle on account creation: auto-generate (unchanged — shown once, never stored anywhere else)
or set a password directly, with the usual confirmation field. The username still always generates
automatically from the domain either way, matching the existing cPanel-style convention.

## Verified against the real routes

Tested through actual HTTP requests against the real login and account-creation endpoints, not a
shortcut: auto mode confirmed unchanged, manual mode confirmed the account was created with the
exact password typed (logged into it for real afterward to prove it), and a mismatched confirmation
was confirmed to correctly reject the whole submission and create nothing at all.
