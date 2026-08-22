# Phase 124 - Off-site backup destinations

The sixth item off the "genuinely zero" list. The backup service had always written backups to local
disk only, deliberately, with an off-site destination documented as a later addition. This is that
addition.

## Local stays authoritative, off-site is a mirror

Every restore, browse and export path reads the local copy only. A destination service fans a
just-completed backup out to every enabled destination as a best-effort, queued push, so a slow
remote transfer never blocks the request or scheduler run that finished the local backup. An
unreachable destination is logged, not thrown, in the same spirit as the scheduled run's "one
account's failure doesn't block the rest."

## Built on the framework's own drivers, not custom transfer code

Three standard filesystem drivers (S3, SFTP and FTP) are all that is added; the framework already
turns an ad-hoc configuration into a working filesystem. S3-compatible providers work through the
same driver with an optional custom endpoint, so no separate driver is needed. The mapping from
the panel's settings to each driver's options is directly tested for all three types, since it is
easy to get backwards and expensive to catch only live.

## New admin page: Backup Destinations

Add, remove, enable and disable, plus a "Validate" button that writes a small marker file, reads it
back and deletes it, proving the destination is reachable and writable before an admin relies on
it. There is deliberately no inline edit form in v1: rotating a credential is remove and re-add, since
partial-update semantics had a real bug for FTP's `ssl` checkbox (an unchecked box reads as false
and would have silently reset it), caught in review before shipping.

## Credentials

Destination settings are stored encrypted using the framework's key-backed cast, and are never
rendered back into the admin form after saving.

## Verified

17 new tests covering configuration translation per type, encryption at rest, failed connectivity
tests, the controller, and push dispatch. Full suite 1,776 and the code-style check green.

## Addendum: root directory must exist before first use (v0.16.60)

Live verification against a disposable SFTP account found that the first connectivity test against a
path prefix not yet created on the remote failed with a confusing message. The drivers create
intermediate directories but not the root itself. Building a destination now creates the configured
root, idempotently. Re-verified for real: the test succeeds against a brand-new directory, and a
real account backup with the destination enabled landed all backup files remotely with matching
SHA-256 checksums. Test resources were cleaned up afterward.
