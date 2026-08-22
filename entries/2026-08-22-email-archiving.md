# Phase 126 - Email archiving

Ninth item off the "genuinely zero" list. The mailbox archiving feature itself was the easy part —
this one's story is really two live-caught mail-routing bugs, one of which took the whole box's mail
down for a few minutes before it got fixed.

## The design: a BCC, not a real mailbox

Every message (incoming, outgoing, or both — independently toggleable) gets silently copied to a
hardcoded internal-only pseudo-address, `store@archive.knjpanel.internal`. It deliberately isn't a
real `Mailbox` row: `sites.account_id` and `mailboxes.site_id` are both `NOT NULL` foreign keys with
`cascadeOnDelete()`, so a clean archive mailbox with no real customer behind it would mean either a
fully fake `Account` (dragging in genuine web-hosting provisioning for no reason) or new nullable
schema just to support one internal address. Instead the archive mailbox lives entirely outside the
app's Eloquent models, as a pair of static config layers on top of the real Postfix/Dovecot setup —
same "layer a static entry alongside the real database-backed one, never touch it" trick this panel
already uses elsewhere (blocked senders/recipients, catch-all). Configurable retention in days (0 =
forever) prunes on a daily scheduled command that just re-applies the currently saved config, since
the config-write action already does the pruning as its last step.

## Bug #1, caught live, caused a real outage: `pcre:` needs a package that isn't installed

The BCC routing used Postfix's `pcre:` map type for `recipient_bcc_maps`/`sender_bcc_maps`. `pcre:`
needs the separate `postfix-pcre` package — not installed on this dev server, and as far as I can
tell not assumed installed anywhere else this panel provisions a mail server. The moment archiving
got enabled, Postfix's cleanup process couldn't evaluate either bcc map for **any** message, and
started bouncing everything — not just the copies meant for archiving — with `451 4.3.0 Error: queue
file write error`. Every account on the box lost mail delivery until this was caught.

Fixed the outage first, properly fixed the bug second: cleared `recipient_bcc_maps`/
`sender_bcc_maps` directly via SSH to restore delivery immediately, confirmed with a real SMTP test
that normal mail flowed again, then only after that root-caused and fixed the actual bug — switched
both maps to `regexp:`, Postfix's own built-in POSIX-regexp table type, which needs no extra package
at all. The pattern in use here is the trivial `/.+/ `, nothing PCRE-specific was ever needed; `pcre:`
was simply the wrong default to reach for.

## Bug #2, caught live: Postfix accepting a message isn't the same as Dovecot delivering it

Even after the `pcre:` fix, archiving still silently lost every copy while normal delivery kept
working fine — no outage this time, just a feature that looked like it was doing nothing.
`virtual_mailbox_domains`/`virtual_mailbox_maps` are Postfix's own "will I accept mail for this
address" tables; actual delivery goes through Dovecot's LMTP agent, which turned out to have its own
**completely separate**, purely SQL-backed `userdb` (`auth-sql.conf.ext` → `dovecot-sql.conf.ext`,
querying `mailboxes`/`sites` directly) with zero awareness of the archive address. `mail.log` showed
Postfix correctly routing the BCC copy to Dovecot's LMTP socket, and Dovecot cleanly bouncing it right
back: `550 5.1.1 <store@archive.knjpanel.internal> User doesn't exist`.

Fixed the same way the Postfix side already was: a static fallback layer, never touching the real
SQL-backed config. Dovecot's `conf.d/*.conf` inclusion (`!include conf.d/*.conf` in the main config)
tries every listed `userdb` block in filename order until one matches, so a new
`conf.d/91-knjpanel-archive.conf` with a `passwd-file`-driver userdb — one line, `user:x:uid:gid::
home::` — sorts in after the existing SQL block and is only ever consulted when SQL finds nothing.
`mail_location = maildir:~` meant the userdb only needed to return the right `home` path (confirmed
`vmail`'s real uid/gid is 5000/5000 via `id vmail`, matching what `dovecot-sql.conf.ext`'s own
`user_query` already hardcodes) — no explicit mail-format override needed. Validated the same way
every other Dovecot-config-touching action in this script already does: `doveconf -n`, backup +
revert on failure, `systemctl reload dovecot` on success.

## Verified

Live, for real, on the actual dev server (not just the 11 tests, which mock `Process`/`Http` and
can't exercise real Postfix/Dovecot binaries — neither bug above would have shown up in them). Sent a
real SMTP message end to end: `mail.log` showed the archived BCC copy landing with `status=sent ...
Saved` (after the earlier bounce, from before the Dovecot fix, in the same log), and `sudo find`
confirmed the message physically present under `/var/mail/vhosts/archive.knjpanel.internal/store/
new/` with fully intact headers. Confirmed the real recipient's own copy was delivered normally the
whole time, including during the `pcre:` outage window itself once mail was flowing again. Ran
`mail-archive-list` and `mail-archive-download` directly against that real archived message — correct
sender/recipient/subject/date parsing and a byte-identical raw `.eml` download. Disabled archiving and
confirmed both bcc maps cleared with `postfix check` staying green.

11 tests (`EmailArchivingServiceTest`, `EmailArchivingTest` feature tests — enable, disable, invalid
retention rejected, config-apply failure surfaces an error, download, non-admin forbidden, local vs.
dispatched-to-linked-Mail-Only-server). Full suite 1,795, `pint --test` green. Both fixes above landed
in a single follow-up commit (v0.16.64) after the initial feature ship (v0.16.63) — the `pcre:` fix
had already been hot-patched directly onto the dev server's disk to stop the outage before the proper
commit/release/redeploy cycle caught up with it.
