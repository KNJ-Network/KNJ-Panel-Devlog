# Phase 59 - The Client Email Section, and a Bug the Whole Session Kept Hiding

Ten items on the account-side Email roadmap. Nine shipped; the tenth is a real, documented wait
on infrastructure that doesn't exist yet — no half-built shortcut taken to close it out anyway.

Mailbox Storage Usage, Bulk Address Import, and Delivery Tracking were the straightforward ones —
real disk usage per mailbox, a CSV importer mirroring KNJ Webmail's own contact importer, and the
existing admin-side delivery log reader reused with a hard domain-suffix check so an account owner
can never see another customer's mail activity through a substring collision. Mail Filters turned
out to be a real gap disguised as a small one: per-mailbox filtering only ever existed behind
Webmail's own login, so an account owner without their mailbox's password couldn't manage it from
the Controller side at all. It's in the Account Panel directly now, with a genuine "global" option
that fans one rule out to every mailbox in a single submission.

Spam Scoring and Greylisting opt-out both needed the same discipline: real, working features that
never touch the shared spamd/postgrey/Postfix configuration every other customer on the server also
depends on. Spam Scoring layers a Sieve rule reading the server's own X-Spam-Level header — an
account can ask for something stricter than the default, never something that reconfigures the
default itself. Greylisting opt-out uses postgrey's own recipient whitelist file, which already
exempted postmaster@ and abuse@ by convention; toggling it for a domain is now the same one-line
addition, fully regenerated from the database on every change. Finding out this needed root write
access to `/etc/postgrey` turned into the same recurring discovery this project keeps making: another
`ProtectSystem=full` gotcha, the fifth or sixth /etc subtree that's had to be added to php-fpm's
ReadWritePaths allowlist the hard way, live, on first real use.

Mailing Lists needed genuinely new infrastructure — its own Postfix lookup table, separate from the
existing forwarders one, so a list address can never collide with (or silently shadow) an existing
forwarder. Verified with a real send: one message to the list address landed in two separate
mailboxes from a single delivery. Mail Encryption keeps the same division of responsibility a real
S/MIME feature needs — the panel generates and holds a self-signed certificate, the account owner
downloads a
password-protected bundle and imports it into their own real mail client, since signing and
decrypting mail is a mail-client feature, not a webmail-server one.

Challenge-response was the one that needed an honest redesign mid-build. The roadmap's own wording
— "require unrecognized senders to confirm" — implies the sender clicks something. Sieve can't call
back into the app to update a database, and the panel never keeps a mailbox's own password around
for a background job to check for replies later, so a sender-driven confirm loop wasn't actually
buildable without either capability. Shipped instead as owner-reviewed approval: unrecognized
senders are held in a Pending folder, never Inbox, until the account owner approves them from the
panel — live-verified with a real stranger's first message landing in Pending, then a second
message from the same address, after approval, landing in Inbox normally.

That last build is also what surfaced the real bug: Dovecot's `lda_mailbox_autocreate` is `no`
server-wide, so every existing filing rule that pointed at a folder which didn't already exist —
Mail Filters' custom rules, Spam Scoring's own Spam folder — was silently failing Sieve execution
and falling back to Inbox, with nothing surfaced anywhere that it had happened. Fixed with Sieve's
own `:create` flag, verified by re-running the exact scenario that first exposed it: a message that
previously landed in Inbox despite a filter rule now lands in the folder the rule actually names.

Nine real features, one real bug found and fixed along the way, and one item left exactly as
Planned as it should be. Next: the DNS-only server project the admin and client roadmaps were
always building toward.
