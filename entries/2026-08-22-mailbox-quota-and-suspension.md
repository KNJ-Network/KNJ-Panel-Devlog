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

## Live-verify addendum: three real bugs, all satellite-related

Same pattern as the Email Disk Usage pass two entries back — live-testing against
`mail.dev.knj.network` (Mail Only satellite) surfaced genuine bugs the unit/feature tests couldn't have
caught, since all three only exist on the cross-server dispatch and passwd-file paths.

**Bug 1 — `mail-dovecot-authmap-write`'s own line validator rejected the new extra field.**
`updateQuota()` threw immediately: "invalid Dovecot auth map line." The provisioning script's
`AUTHMAP_LINE_RE` — the regex gating every line written into `/etc/dovecot/mail-users`, a root-owned
auth file — required the trailing extra-fields column to be exactly empty (`::$`), predating this
feature. Fixed by extending the pattern to also accept `(quota_rule=\*:bytes=[1-9][0-9]*)?` — anchored
to Dovecot's own quota_rule shape, not free-text, since this still feeds a privileged file.

**Bug 2 — quota silently didn't enforce even once the line was accepted.** With bug 1 fixed, a real
LMTP delivery of a 2MB message against a 1MB quota was accepted and saved to INBOX — no rejection, no
error, `maildirsize`'s limit line read `0S` (unlimited) despite `quota_rule=*:bytes=1048576` being
present in the file. Root cause: `/etc/dovecot/mail-users` is read by *both* the passdb and userdb
`passwd-file` stanzas (see `auth-passwdfile.conf.ext`) — an unprefixed extra field on a shared file is
ambiguous between the two, and Dovecot silently accepted the line without ever surfacing it to the
quota plugin. The fix is Dovecot's own documented convention for this exact situation: prefix fields
meant only for userdb with `userdb_`. Changed `MailAuthMapService` to emit `userdb_quota_rule=` instead
of `quota_rule=`, and widened the line validator's regex to match. `dovecot-sql.conf.ext`'s own
`user_query` was unaffected and needed no change — `password_query`/`user_query` are already two
separate, unambiguous queries there, so the bare column name was correct all along on that path.

**Confirmed for real, end to end**, after both fixes: created a disposable mailbox, set its quota to
1MB, sent a 2MB message through the actual Postfix→Dovecot LMTP path (not a CLI shortcut) — accepted
(first message under a fresh mailbox is always allowed regardless of size, standard LDA/LMTP behavior).
Sent a second, trivial message immediately after — rejected: `552 5.2.2 Quota exceeded (mailbox for
user is full)`, bounced by Postfix exactly as a real client would see it. `maildirsize`'s limit line
correctly read `1048576S` throughout. Suspension confirmed separately via the auth map's own exclusion
logic (no code path difference from the SQL side's `WHERE m.suspended = 0`, so no new live-only
behavior to catch there). Disposable mailbox and test messages cleaned up afterward.
