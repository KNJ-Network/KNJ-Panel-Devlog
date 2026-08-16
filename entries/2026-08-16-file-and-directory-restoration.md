# Phase 86 - Restoring the One File, Not the Whole Account

Backup Restoration has always meant one thing: wipe the account's home directory and re-extract
the whole files archive. That's correct but blunt, and per-file and per-directory restoration
needed a different mechanism rather than a smaller version of the existing one.

## Browsing an archive without a subprocess for every click

Listing what's inside a backup archive doesn't need elevated rights. Each level of the browse view
is built by inferring directories from deeper archive members rather than trusting the archive's
own directory entries, which is robust whether or not an explicit entry exists for a folder.

## Additive, not wipe-and-replace

The full restore replaces the account's files, which is right for "put this account back exactly as
it was" and wrong for "I overwrote one config file". The new file restore extracts only the named
file, or everything under a named folder, into place with correct ownership, touching nothing
else.

The one new input here is a path coming from the web request, so it is validated independently on
both the web side and the provisioning side, as with every other user-influenced argument since
audit #07.

## What shipped

A "Browse" link next to every completed account backup, on both admin and account-owner sides, with
breadcrumb navigation through the account's file tree and a "Restore file" or "Restore folder"
button on every entry. Each has its own confirmation spelling out what will and won't change.
