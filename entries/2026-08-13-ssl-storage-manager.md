# Phase 76 - A Certificate Repository Instead of a Site List

The next item on the gap list: an SSL Storage Manager. The existing SSL page already listed every
site's current certificate, but that's exactly its limit — it only ever knows about a certificate
still attached to a site that still exists. A certificate left behind after its domain was removed,
or a replaced certificate whose old files never got cleaned up, was invisible to the panel entirely.
This is the other half: a repository of every certificate this server actually has on disk,
independent of what — if anything — is using it.

## Reading the filesystem instead of the database

The privileged provisioning script gained a new read-only action that walks both certificate
locations this server writes to (Let's Encrypt's own store and the panel's own custom-upload store)
and runs each one through `openssl x509` to pull its subject, issuer, expiry, and fingerprint. The
panel app itself cross-references the results against its own `sites` table afterward — the
provisioning script's job is only ever to report what's really on disk, never to reach into the
database on its own.

Every row on the new page ends up in one of three states: attached to a site currently using it,
still the panel's own service certificate (protected — deleting that would take the panel itself
offline), or orphaned. Only an orphaned entry ever gets a delete button, and the delete request is
re-checked against the same live cross-reference before anything is removed — never trusted from
whatever the page happened to be showing when the click landed.

## A shell bug only a real certificate would show

Wrote this, tested it thoroughly against faked output, shipped it — and it broke immediately
against the one real certificate on the server. `openssl x509 -fingerprint -sha256`'s own label
turned out to be `sha256 Fingerprint=`, lowercase, not `SHA256 Fingerprint=` as every reference
example (including the ones this was written against) shows it. A `grep` that finds no match exits
non-zero, and this script runs under `set -e` — so the one field that didn't match didn't just come
back blank, it took the whole action down with it, for every certificate, every time. Fixed by
matching case-insensitively and letting each field fail independently rather than the whole action
failing on one. A reminder that "openssl -text output" isn't as fixed a format as it looks from a
few examples — the only way to know for certain is a real certificate on the real server.

## Next

An admin database root password reset, then per-database access profiles.
