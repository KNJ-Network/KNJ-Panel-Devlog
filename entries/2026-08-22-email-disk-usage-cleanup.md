# Phase 122 - Email Disk Usage cleanup

Third item off the "genuinely zero" list. Mailbox Storage (`Account/MailboxUsageController`) already
showed per-mailbox usage — that part shipped a while back — but it was purely a display page. cPanel's
own Email Disk Usage tool goes one step further: it lets the owner clear out what's actually eating the
space, specifically Trash and Junk, without going into webmail folder-by-folder.

## Scoping against the real cPanel feature, not a guess

Checked what cPanel's tool actually does before building anything: it's not a folder-picker/date-range
purge UI — it's narrow, essentially just "Empty Trash" and "Empty Junk" per mailbox. That's the exact
scope this pass targets, deliberately smaller than a full IMAP-style bulk-delete tool.

## Why doveadm, not IMAP

The account's own webmail client (`ImapMailboxClient`) has per-message delete/move but nothing bulk,
and more importantly, admin-side cleanup can't authenticate as the mailbox at all — this panel never
keeps a mailbox's own password around after creation (documented on `MailUsageService` already, for
the same reason the usage report itself has to run as root against the maildir directly rather than
logging in over IMAP). So the existing `mail-usage-report` provisioning action's pattern — root reads
the maildir directly, no credential needed — is what a purge action had to follow too.

`doveadm expunge -u <mailbox> mailbox <folder> all` is the right primitive: it goes through Dovecot
itself, so its own index/cache files stay consistent, unlike a raw `rm -rf` on the maildir which would
leave stale IMAP UIDs and unread counts behind. `doveadm` was already trusted and already run as root
in this script (for `doveadm pw` at mailbox creation) — this is the same privilege level, just a
different subcommand.

## Trash/Junk sizes, not just a purge button

Extended `mail-usage-report` to also report `.Trash`/`.Junk` byte counts per mailbox (Maildir++ layout
makes these predictable sibling directories under the mailbox root — just two more `du -sb` calls
alongside the existing total), so the UI shows "Trash: 1.2 MB — Empty" rather than a blind button. The
Empty button only appears when there's actually something in that folder.

## Safety

The folder name is allowlisted to exactly `Trash`/`Junk` at both layers — Laravel validation
(`Rule::in(['Trash', 'Junk'])`) and the provisioning script's own argument check — since it feeds
straight into a root-run doveadm command; free-text here would be a real privilege-escalation surface,
not just a UX nicety. Ownership is checked the same way `AutoresponderController` already does for
per-mailbox actions (`abort_unless($mailbox->site->account_id === $account->id, 404)`), and the button
itself goes through the panel's own `data-confirm` modal, not a native `confirm()`.

## Verified

9 new tests (5 `MailUsageService` — trash/junk parsing, backward-compatible defaulting for the old
2-field report format, `purgeFolder()` calling the script correctly, rejecting an unsupported folder
without ever calling the script; 4 `MailboxUsageControllerTest` — the breakdown renders, an owner can
empty their own Trash, an unsupported folder is rejected server-side even if a request bypasses the
UI, and an owner can't purge another account's mailbox). Full suite 1,744 (up from 1,736), `pint --test`
green.
