# Phase 11 - Closing Out the Audit Follow-Ups

Yesterday's [security audit](2026-07-25-first-security-audit.md) surfaced a handful of findings
that were fixed same-day, plus three items flagged as follow-up work rather than quick patches.
All three are done now.

## A dedicated account for the app itself

The panel's own backend process had been running under the login the dev server was originally set
up with — convenient on a development box, but not appropriate for something meant to be installed
by someone else. The app now runs under its own dedicated, unprivileged service account with
narrowly-scoped permissions, separate from anyone's personal login. Verified by creating and
removing a real test hosting account end to end under the new setup.

## Resellers are now scoped

The Admin/Reseller/Account role split has existed since the beginning, but the Reseller role
hadn't yet been narrowed relative to Admin. Fixed: Admins create Reseller accounts, a Reseller only
sees and manages the hosting accounts assigned to them, and server-level sections are Admin-only.
Covered by isolation tests, including attempts to reach another Reseller's accounts or admin-only
pages directly, all of which are rejected.

## A deployment process improvement

While making the change above, we found that the dev server's copy of the code and the
version-controlled history had drifted apart. Nothing was lost. Deployment is now one-directional:
everything goes through version control first, and the server only pulls from there.

## Also: a sidebar reorganization

Mail settings and service health checks moved out of "Security" into their own "Services"
section. A small unused mail listener was also switched off as tidy-up.
