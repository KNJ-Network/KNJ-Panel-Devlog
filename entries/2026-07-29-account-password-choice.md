# Phase 21 - Choosing Your Own Password on Account Creation

A small but useful gap: Create Account has always auto-generated the owner's password and shown it
once, which is a sensible default — but there was no way to set one explicitly, which matters when
handing an account to a client who wants to pick their own from the start.

## What shipped

A toggle on account creation: auto-generate (unchanged — shown once) or set a password directly,
with the usual confirmation field. The username still generates automatically from the domain
either way.

## Verified against the real routes

Tested through real HTTP requests against the login and account-creation endpoints: auto mode
unchanged, manual mode created the account with the exact password typed (and logged in with it),
and a mismatched confirmation rejected the whole submission and created nothing.
