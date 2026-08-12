# Phase 16 - The DNS Roadmap, Closed Out

DNS has been "working on this first" since the very start of this build log. As of today every
item on that original list is either shipped or a deliberate, documented "not yet" — the arc is
closed.

## Inline bulk-save zone editing

Every record in a zone is now editable in place, with one "Save All Records" button submitting
the whole table as a single all-or-nothing batch — one bad row rejects the entire save rather than
partially writing a broken zone, and it's one zone-file regeneration per save instead of one per
row. Caught a real pre-existing bug along the way: the record-name validator rejected any name
starting with an underscore, which would have silently blocked editing real records like
`_dmarc`/`_domainkey` through either the old single-record editor or the new bulk one.

## Restore Default Records

Self-service recovery for an account owner who deletes some or all of their zone's records —
by accident or otherwise — and breaks their own site or mail. It's additive only: it checks every
expected record individually and creates only what's actually missing, never touching or
duplicating anything already there, including a customer's own hand-added records, and reuses
existing DKIM key material rather than rotating it. Tested against the exact failure it's meant to
fix: wiped every record on a test zone, confirmed the empty-state warning appeared, restored in
one click, confirmed every record was back and the zone still validated.

## Per-zone DNSSEC

Real key generation and signing through BIND itself, not a UI checkbox — this app never handles
DNSSEC key material directly. One click enables or disables signing per zone, and the DS record
values needed at the registrar are shown once signing completes. Two real infrastructure bugs only
surfaced by actually testing signing live, not from documentation: a permissions gap that stopped
BIND writing its own signed-zone files, and Ubuntu's stock AppArmor profile for BIND hard-coding
part of its own config directory read-only regardless of the Unix permissions actually set.
Confirmed working with a real external DNSSEC query against a real signed domain, not just
checking the database.

## DNS Clustering (master-side readiness)

Real BIND primary/secondary replication — authenticated zone transfers and change notifications —
not a custom sync protocol. This isn't the standalone DNS-only server product itself; it's what
this server needs to be ready to authorize and notify one the moment that product exists. Adding a
DNS-only server generates a real cryptographic key and the matching BIND configuration
automatically. Verified twice: once against BIND directly before any application code was
involved, confirming a zone transfer is refused without the key and succeeds, fully authenticated,
with it — then again through the real interface end to end.

## Addon domains (Park a Domain)

Account-side self-service: add another domain to an existing hosting account, with its own web
config, home directory, DNS zone, and SSL, sharing the account's existing system user and
database. Surfaced a real gap in account deletion along the way — destroying an account only ever
cleaned up its *primary* domain's configuration, leaving any addon domains orphaned. Fixed
alongside the feature itself, not deferred.

## Email Routing, and backfilling older domains

Per-domain control over whether this server or a remote system handles a domain's mail (Email
Routing), plus a way to retroactively add SPF/DKIM/DMARC to any domain
created before that automation existed — additive, so it never touches records a customer already
has of their own.

## What's left

Genuinely nothing planned is undone. The one remaining item — a manual "push everything now" DNS
sync trigger — is correctly sequenced *after* a standalone DNS-only server product exists to
actually test a resync against, rather than being built speculatively ahead of it.
