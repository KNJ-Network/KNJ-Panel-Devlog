# Phase 98 - First-Run Setup Wizard

This phase looked at how established hosting panels handle installation and first login, and built
a proper onboarding flow of our own.

## What was missing

Our install script had no legal-agreement step, and its first-admin flow, which generated a random
password and printed it once at the end of the install log, wasn't a real onboarding experience:
no agreement to accept, and no choice of admin email or password.

## The wizard

The install script now generates a one-time setup link, printed at the end of the install instead
of credentials. Visiting it walks through:

1. **Legal agreements**: the Terms of Service and Privacy Policy, with a mandatory "I agree"
   checkbox.
2. **Admin account**: the operator sets their own email and password. Nothing is auto-generated.
3. **Basic config**: hostname, timezone, and contact email, using the same validation as the
   Server Settings page.
4. **DNS-only role only**: an optional step to link the server to a Main server right there, with a
   "skip, link later" option.
5. **Congratulations**: a button to the login page, where the chosen credentials work.

The wizard is gated by a completion flag rather than by whether an admin exists, since the admin is
created partway through and gating on that would lock the operator out of the remaining steps. Setup
links stop working once setup is complete, apart from the static final page, so a refresh doesn't
break.

## A bug caught in testing

Four of the five wizard views forgot to pass the link token through to their templates. It only
surfaced once the tests rendered each view rather than following redirects past it.

## Verified

The full suite (1,480 tests) and code-style checks pass, with new coverage for invalid links, the
completed flag, the legal-acceptance gate, password hashing and acceptance timestamping, the
DNS-only role step, and skip-linking. A full run on a fresh machine followed, and its findings are
covered in the next phase.
