# Phase 106 - Mail Only: Syncing Existing Accounts to a New Mail Server

## The gap behind "existing mailboxes aren't copied over yet"

Stage 2's Mail Settings selector shipped with an honest caveat in its own UI: switching to a
linked Mail Only server didn't do anything for mailboxes that already existed — only new activity
would route there. Digging into why surfaced something bigger than a missing resync loop: Postfix
and Dovecot don't get told "does this mailbox exist" or "what's its password hash" through the
provisioning script at all. On Main, both query the panel's own MySQL database live —
`dovecot-sql.conf.ext`'s `password_query`/`user_query`, and three separate `virtual_mailbox_*`
Postfix maps, all pointed straight at `mailboxes`/`sites`. A genuinely separate Mail Only box has
no local copy of that database worth querying. Even a "successful" dispatched `mailbox-create`
would have left a maildir on disk with nothing anywhere that could ever authenticate against it.

## Same pattern as DNS, applied one level deeper

The fix follows the same shape as everything else built this week: never share a live database
connection across the link, push generated config instead. `MailAuthMapService` builds two flat
files from Main's current mailbox/domain state — a Postfix `hash:` map (`virtual_mailbox_domains`
+ `virtual_mailbox_maps`) and a Dovecot passwd-file (auth + home/uid/gid lookup in one file, one
line per mailbox) — and pushes both through two new provisioning actions
(`mail-virtual-maps-write`, `mail-dovecot-authmap-write`) over the exact same dispatcher built in
Stage 2, no new transport needed.

It always rebuilds the *full* current state, never incrementally — the same convention DNS's own
zone-membership push already uses. That single design choice is what actually closes the original
gap: calling `regenerate()` once a server is selected replays every mailbox that already existed,
not just ones created afterward, with no separate "bulk sync job" concept needed at all. It's also
wired into every mailbox create/delete/password-change directly, so a linked server's map stays
current as things change afterward, not just at the moment of switching.

Correctly a no-op everywhere else: `regenerate()` checks `mail.active_server_id` itself before
doing anything, so an install that's never touched the selector pays nothing — no extra local
`postmap`/`doveconf`/service-reload calls on every mailbox action, matching the "zero behavior
change until an admin opts in" bar every other piece of this feature has held to.

## The Postfix/Dovecot side — new ground, not yet proven

`setup-mail-server.sh` gained a `mail_only`-specific branch: instead of the SQL auth config Main's
install writes, it points Postfix at the two pushed `hash:` files and switches Dovecot from
`auth-sql.conf.ext` to a new `auth-passwdfile.conf.ext` (handled for both Dovecot 2.3 and 2.4
syntax, matching this script's existing dual-version handling elsewhere). This is genuinely new
systems configuration — not something covered by the existing 1593 PHPUnit tests, which only
exercise the PHP side. It follows the same backup-before-write, `postfix check`/`doveconf -n`
validate, revert-on-failure discipline every other config-writing provisioning action already
uses, and both scripts pass `bash -n`. But it hasn't run against a real Dovecot/Postfix yet — that
needs the same real linked Mail Only box Stage 1 and 2's own live verification is waiting on.

## One more thing found along the way

Server Setup's deferred certificate-issuance flow (`issue-panel-cert`, triggered once a domain is
configured after a self-signed-only bootstrap) was unconditionally running *both*
`setup-mail-server.sh` and `setup-dns-server.sh` regardless of role — a pre-existing gap that
predates this session's work, not something introduced by it. A DNS-only box setting a hostname
this way would have gotten Main's full authoritative DNS setup on a box that's supposed to stay
secondary-only. Fixed alongside this pass since it's the same code path the new `mail_only`-aware
mail setup needed to go through correctly: now reads the install's actual role from its own
`.env` (the same `PANEL_ROLE=` line `bootstrap-server.sh` already writes) and only runs each
script for the roles it actually applies to.

## Explicitly not done

Forwarders, catchalls, and mailing lists (`virtual_alias_maps`) have the exact same live-SQL
problem this closes for mailboxes — a Mail Only box's `mail_forwarders` table is just as empty as
its `mailboxes` table was. Not touched in this pass; a forwarder created on Main today simply
won't route through a linked Mail Only server until this gets the same flat-map treatment as a
separate follow-up. The account-sync UI copy and this entry both say so explicitly, matching this
session's own standing "verify before claiming gaps are closed" discipline.

## Verified

Full local suite (1597 tests) and `pint --test` both green. New coverage: `MailAuthMapServiceTest`
(no-op with no active server, correct flat-map content generated and dispatched, remote-routed
domains excluded, dispatch failures surfaced as exceptions). Both modified shell scripts pass
`bash -n`. No live VM test yet — pending the real linked Mail Only box currently being prepared.
Committed and pushed as `c9c2a7f`.
