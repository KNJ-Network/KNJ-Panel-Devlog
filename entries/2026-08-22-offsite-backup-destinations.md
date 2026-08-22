# Phase 124 - Off-site backup destinations

Sixth item off the "genuinely zero" list. `BackupService` has always written backups to local disk
only — deliberately, per its own docblock, with "an off-site/remote destination is a deliberate later
addition, not an oversight" — and this pass is that addition.

## Local stays authoritative, off-site is a mirror

Every restore, browse, and export path in `BackupService` reads the local copy only; that never
changes. A new `BackupDestinationService` fans a just-completed backup out to every enabled
destination as a best-effort push, queued (`PushBackupToDestinationsJob`) rather than inline, since a
remote transfer has no business blocking the request or scheduler run that just finished the local
backup. A destination being unreachable is logged, not thrown — same shape as
`BackupService::runScheduled()`'s own "one account's failure doesn't block the rest of the run"
handling, just one layer further out.

## Built on Laravel's own Flysystem drivers, not custom transfer code

`league/flysystem-aws-s3-v3`, `league/flysystem-sftp-v3`, and `league/flysystem-ftp` are the three new
dependencies — Laravel's `Storage::build($config)` already knows how to turn an ad-hoc config array
into a working S3/SFTP/FTP filesystem with zero custom adapter code. S3-compatible providers
(Backblaze B2, Wasabi, MinIO, etc.) come for free from the same `s3` driver via an optional custom
endpoint — no separate driver needed, matching how this panel prefers one shared mechanism over
several near-duplicate ones wherever the underlying protocol is genuinely the same shape.

`BackupDestinationService::translateConfig()` — the small pure mapping from this panel's own config
keys (`path_prefix`, etc.) to whatever key names each Laravel driver actually expects (`root`, etc.) —
is deliberately `public`, not private: it's exactly the kind of thing that's easy to get backwards
(pointing a destination at a bucket's root instead of its configured prefix) and expensive to catch
only via a live test, so it's directly asserted in `BackupDestinationServiceTest` for all three types
rather than only exercised indirectly.

## New WHM page: Backup Destinations

Add/remove/enable/disable, plus a "Validate" button that writes a small random-content marker file,
reads it back, and deletes it — proving the destination is actually reachable and writable before an
admin relies on it for something as important as an off-site backup copy. Deliberately no inline edit
form for v1: rotating a credential is remove-and-re-add, not a partial-update form, since a partial
update's "leave blank fields unchanged" semantics turned out to have a real bug for FTP's `ssl`
checkbox specifically (an unchecked checkbox reads as `false`, not "unspecified," so a partial update
would have silently reset `ssl` to off on every edit that didn't explicitly re-check it) — caught during
review before it ever shipped, not worth the complexity for a v1 credential-rotation path that
remove-and-re-add already covers completely.

## Credentials

`BackupDestination.config` is `encrypted:array` at the model layer (Laravel's own `APP_KEY`-backed
cast, same pattern already used for `Server`, `Webhook`, and several others in this codebase) — never
stored in plaintext, never rendered back into the admin form after save.

## Verified

17 new tests: 6 `BackupDestinationServiceTest` (S3/SFTP/FTP config translation asserted exactly per
type, encrypted-at-rest confirmed by reading the raw DB column, a failed connectivity test recorded
without throwing), 8 `BackupDestinationControllerTest` (CRUD, per-type validation, admin-only access,
credentials never rendered back to the page), plus 2 new `BackupServiceTest` cases confirming the
push job is dispatched only when at least one destination is enabled. Full suite 1,776 (up from 1,759),
`pint --test` green.

## Addendum: root directory must exist before first use (found live, fixed in v0.16.60)

Live-verifying against a real disposable SFTP account (`knjbackuptest@mail.dev.knj.network`) surfaced
a real bug: the first `test()` against a destination whose `path_prefix` hadn't been created on the
remote yet failed with a confusing `Wrote a test file but read back different content.` rather than a
clear directory-not-found error. Root cause: Flysystem's SFTP/FTP adapters auto-create *intermediate*
directories for paths written below an existing root, but never the root itself — a fresh
`path_prefix` a customer hasn't manually pre-created on their bucket/server was silently broken.

Confirmed via SSH that no `backup-test` directory existed on the target before the failure, and that
writing directly into the account's home directory (empty `path_prefix`) succeeded immediately,
isolating the bug to root-creation rather than credentials or connectivity.

Fix: `BackupDestinationService::buildDisk()` now calls `$disk->createDirectory('')` on the already
root-scoped Flysystem disk right after `Storage::build()` — idempotent, so harmless to call on every
`test()`/`push()`, not just first setup. A new unit test (`test_build_disk_creates_the_configured_root_directory_before_use`)
asserts this is actually attempted by reflecting into `buildDisk()` and confirming the connection
attempt itself is what surfaces, not a silently-skipped step.

Re-verified for real after the fix shipped (v0.16.60): `test()` against a brand-new, never-created
remote directory now succeeds and the directory is confirmed created server-side. Went further and
triggered an actual account backup with the destination enabled — the queued push landed all 5 backup
files on the remote SFTP target, and every file's SHA-256 checksum matched its local counterpart
exactly, proving the end-to-end byte-identical mirror this feature exists to deliver, not just
connectivity. All disposable test resources (the `knjbackuptest` system user, its scoped
`sshd_config.d` password-auth override, the test `BackupDestination` row, and the local/remote test
backup files) were cleaned up afterward.
