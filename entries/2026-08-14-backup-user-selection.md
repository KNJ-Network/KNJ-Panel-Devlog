# Phase 84 - The Smallest Gap on the List

Last open item under Backup & Server Status: which accounts actually get backed up on the
automated scheduled run. Today's answer was "all of them, always" — every active account, every
scheduled run, no way to opt one out short of suspending it.

## One column, one filter

The whole feature is a single nullable-free boolean: `excluded_from_scheduled_backups` on
`accounts`, defaulting to false so nothing changes for an account until an admin actually touches
this new page. `BackupService::runScheduled()`'s existing account loop grows one more `where()` —
the on-demand path (`backupAccount()` called directly, whether from the admin's own Backups page or
the account owner's) never goes near this column at all, so a single account backup still works
exactly the way it always has regardless of the scheduled-run setting. Deliberately not the same
lever: an admin excluding an account from the nightly run isn't a statement about whether that
account should ever be backed up, just about whether it needs to happen automatically tonight.

## What shipped

A new WHM admin page, Backup User Selection, next to the existing Backup Settings — every account
listed with a checkbox reflecting its current inclusion state, one form, one save. Closes out
Backup & Server Status.

## Next

Continuing down the gap list.
