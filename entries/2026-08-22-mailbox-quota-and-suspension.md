# Phase 123 - Per-mailbox quota and suspension

Fourth item off the "genuinely zero" list. `quota_mb` has existed on the `mailboxes` table since the
mail feature first shipped, but it was never more than a stored number: set once at mailbox creation
from the account's global default, displayed read-only on the Email Disk Usage page, never actually
enforced by Dovecot and never editable afterward. Mailbox suspension didn't exist at all — no column,
no service method, nothing.

## Two auth backends, two places to enforce both things

This panel's Dovecot auth has always run one of two ways depending on topology (see `MailAuthMapService`'s
own doc comment): on Main (or any install with no Mail Only satellite selected), Dovecot's SQL passdb/
userdb queries the local `mailboxes`/`sites` tables directly — same MySQL database the Laravel app itself
writes to, no round trip needed. A linked Mail Only satellite has no local copy of that data worth
querying, so `MailAuthMapService` instead pushes the full current mailbox set as a flat Dovecot
passwd-file (`user:hash:uid:gid::homedir::extra`) every time anything mail-related changes.

Both quota and suspension had to be wired into both paths:

- **SQL path** (`dovecot-sql.conf.ext.2.3.template`): `password_query` gained `AND m.suspended = 0` —
  a suspended mailbox simply doesn't come back from the query, so Dovecot's own passdb lookup fails
  with "user unknown", not a special suspended-specific error path. `user_query` gained a `quota_rule`
  extra column, `CASE WHEN m.quota_mb > 0 THEN CONCAT('*:bytes=', m.quota_mb * 1024 * 1024) ELSE NULL
  END` — Dovecot's quota plugin recognizes `quota_rule` as a userdb extra field automatically, and a
  `NULL` column is treated as absent, so `quota_mb = 0` stays genuinely unlimited rather than emitting
  a zero-byte rule.
- **Passwd-file path** (`MailAuthMapService::writeDovecotAuthMap()`): suspended mailboxes are filtered
  out of the collection before the file is built — same "no row, no login" outcome as the SQL side's
  `WHERE` clause, not a different mechanism. A `quota_rule=*:bytes=N` extra field is appended to the
  line only when `quota_mb > 0`, mirroring the SQL template's own `CASE`.

Deliberately never marking a row "suspended" *and* still listing it somewhere with a deny flag — leaving
it out of both auth sources entirely means there's exactly one thing to get right (the filter), not two
in agreement.

## Enabling the quota plugin itself

Dovecot's quota plugin isn't installed by a separate package (unlike Sieve, which needed
`dovecot-sieve`) — it ships in `dovecot-core` and just needs turning on. Added to the same fully-managed
`conf.d/90-knjpanel-settings.conf` drop-in `mail-server-config-write` already regenerates (the file this
script's own comments describe as "Dovecot's own supported mechanism for local overrides, never touching
the stock files"): `quota` added to LMTP's `mail_plugins` (the delivery-time enforcement point),
`imap_quota` added to IMAP's (so a client can query its own quota — visibility, not enforcement), and a
`plugin { quota = maildir:User quota }` block. Maildir++'s own `maildirsize` file handles the accounting
— no separate quota database or service to run, matching this panel's general preference for
zero-extra-moving-parts primitives wherever Dovecot already has one built in.

## What's account-side, not WHM

Both quota and suspension are self-service on the Email Accounts page (`Account/MailController`), not a
new admin/WHM surface — this panel currently has no WHM screen that manages individual mailboxes at all,
and cPanel's own "Manage Mailbox Quota" is genuinely account-side too (an owner adjusting how much of
their own allotment one address gets). Suspension reads oddly self-service at first (why would an owner
lock themselves out?) until framed the way cPanel's own "Suspend Login" is: a defensive tool for a
compromised mailbox — lock out login while investigating, without losing a single message, since mail
keeps arriving via LMTP the whole time.

## Safety

`updateQuota()` rejects negative values before ever touching the database row. The account-side
controller enforces the same ownership check every other per-mailbox action in this file already uses
(`abort_unless($mailbox->site->account_id === $account->id, 404)`). Neither quota nor suspension touches
the provisioning script at all — both are pure database writes plus `MailAuthMapService::regenerate()`
(a no-op on any install with no Mail Only server active), so there's no new privileged action, no new
argument-injection surface.

## Verified

14 new tests (6 `MailboxServiceTest` — quota update, negative-quota rejection, zero-means-unlimited,
suspend/unsuspend, confirming neither ever touches the provisioning script; 6
`MailboxQuotaSuspensionTest` — an owner can update their own quota, negative quota is rejected, an owner
can't touch another account's mailbox quota or suspension state, suspend/unsuspend round-trips, guest
redirect; 2 `MailAuthMapServiceTest` additions — suspended mailboxes excluded from the pushed passwd-file
entirely, quota_rule appended only for a capped mailbox). Full suite 1,759 (up from 1,745), `pint --test`
green.
