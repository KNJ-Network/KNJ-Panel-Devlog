# Phase 84 - The Smallest Gap on the List

The last open item under Backup & Server Status: which accounts get backed up on the automated
scheduled run. Previously the answer was "all of them, always", with no way to opt one out short
of suspending it.

## One column, one filter

The whole feature is a single flag on the account record, defaulting so that nothing changes for an
account until an admin touches the new page. The scheduled run grows one more filter. The on-demand
path, a single-account backup from either the admin or the account owner, never uses the flag, so
a manual backup still works regardless. This is deliberate: excluding an account from the nightly
run isn't a statement about whether it should ever be backed up, just whether it needs to happen
automatically tonight.

## What shipped

A new admin page, Backup User Selection, sits next to Backup Settings. Every account is listed with
a checkbox reflecting its current inclusion state, with one form and one save. This closes out
Backup & Server Status.
