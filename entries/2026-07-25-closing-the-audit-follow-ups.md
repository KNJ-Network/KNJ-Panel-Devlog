# Phase 11 - Closing Out the Audit Follow-Ups

Yesterday's [security audit](2026-07-25-first-security-audit.md) surfaced a handful of findings
that were fixed same-day, plus three items flagged as real follow-up work rather than quick
patches. All three are done now.

## A dedicated account for the app itself

The panel's own backend process was running under the same login the server was originally set
up with — convenient for a single developer's own box, but not something that holds up once this
is meant to be installed by someone else on their own server. The app now runs under its own
dedicated, unprivileged service account, with its own narrowly-scoped permissions, completely
separate from anyone's personal login. Verified by creating and then removing a real test hosting
account end to end under the new setup — system user, web server config, database, file
ownership, all of it — before trusting it.

## Resellers are now actually scoped

The panel's role system has had an Admin/Reseller/Account split since the very beginning, but the
Reseller role had never actually been wired up to *mean* anything narrower than Admin — it was
functionally a second Admin account. Fixed properly: Admins now create Reseller accounts, a
Reseller only ever sees and manages the hosting accounts assigned to them, and server-level
sections (infrastructure management, security tooling) are Admin-only. Covered by a full set of
isolation tests — a Reseller genuinely cannot see or touch another Reseller's accounts, and
several attempts to spoof around that (submitting someone else's ID directly, hitting admin-only
URLs directly) are all rejected.

## A deployment process gap, caught and fixed

While making the change above, we found that the server's copy of the code and the version-
controlled history had quietly drifted apart — commits were being made and pushed correctly, but
the running server wasn't actually being kept in sync with them the way it was supposed to be.
Nothing was lost (checked carefully before touching anything), but it was closer than it should
have been. Deployment is now strictly one-directional: everything goes through version control
first, the server only ever pulls from there, and that's now documented as a hard rule rather
than an assumption.

## Also: a sidebar reorganization and one more hardening item

Mail settings and service health checks have moved out of "Security" and into their own
"Services" section, with room reserved for a couple of settings pages not built yet. And an
unused, unencrypted mail protocol listener — already blocked by the firewall, so not an actual
exposure, but unnecessary all the same — has been switched off.

Back to feature work next.
