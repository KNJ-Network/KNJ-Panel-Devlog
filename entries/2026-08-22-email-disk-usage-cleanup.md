# Phase 122 - Email Disk Usage cleanup

The third item off the "genuinely zero" list. Mailbox Storage already showed per-mailbox usage but
was purely a display page. A comparable panel's Email Disk Usage tool goes one step further, letting
the owner clear out what is eating space, specifically Trash and Junk, without going through webmail
folder by folder.

## Scoping against the real feature, not a guess

That tool is narrow: essentially "Empty Trash" and "Empty Junk" per mailbox. That is the scope here,
deliberately smaller than a full bulk-delete tool.

## Why the mail server's own tool, not IMAP

Admin-side cleanup cannot authenticate as the mailbox, because the panel never keeps a mailbox's
password after creation. The purge therefore uses the mail server's own supported tooling, which
keeps its index and cache files consistent, unlike a raw file delete that would leave stale message
numbers and unread counts behind.

## Trash/Junk sizes, not just a purge button

The usage report now includes Trash and Junk sizes per mailbox, so the UI shows "Trash: 1.2 MB,
Empty" rather than a blind button. The button appears only when there is something to empty.

## Safety

The folder is restricted to Trash or Junk at both the application and the provisioning layers,
ownership is checked as for other per-mailbox actions, and the button uses the panel's own
confirmation modal.

## Verified

9 new tests; full suite 1,744 and the code-style check green.

## Live-verify addendum: two real bugs, both satellite-related

Testing against a disposable mailbox on a linked Mail Only satellite surfaced two bugs that only
exist on the cross-server path:

**Bug 1: stale satellites.** Trash/junk sizes stayed at zero because both linked test satellites
were several releases behind; our update step for them had been skipped for a few releases. It now
runs after every release.

**Bug 2: the new action was not on the satellite's accepted-action list.** With satellites current,
the purge still failed with an empty error. Fixed in v0.16.55 with a regression test confirming the
action is reachable through the real dispatch route.

With both fixed, the full loop was re-run for real: usage showed the expected sizes and purging
Trash and Junk emptied them. The disposable mailbox was cleaned up afterward.
