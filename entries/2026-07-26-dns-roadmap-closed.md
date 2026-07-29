# Phase 16 - The DNS Roadmap, Closed Out

DNS has been "working on this first" since the start of this build log. As of today every item on
that original list is either shipped or a deliberate, documented "not yet".

## Inline bulk-save zone editing

Every record in a zone is now editable in place, with one "Save All Records" button submitting the
whole table as a single all-or-nothing batch — one bad row rejects the entire save rather than
partially writing a broken zone, and it's one zone-file regeneration per save. Caught a real bug
along the way: the record-name validator rejected names starting with an underscore, which would
have blocked editing real records like `_dmarc`/`_domainkey`.

## Restore Default Records

Self-service recovery for an account owner who deletes some or all of their zone's records and
breaks their own site or mail. It's additive only: it checks every expected record individually and
creates only what's missing, never touching anything already there, including a customer's own
hand-added records, and reuses existing DKIM key material rather than rotating it. Tested against
the exact failure it's meant to fix: wiped every record on a test zone, restored in one click, and
confirmed the zone validated.

## Per-zone DNSSEC

Real key generation and signing through BIND itself, not a UI checkbox. One click enables or
disables signing per zone, and the DS record values needed at the registrar are shown once signing
completes. Two infrastructure issues only surfaced by testing signing live on a development
server, and were fixed. Confirmed with a real external DNSSEC query against a signed test domain.

## DNS Clustering (primary-side readiness)

Real BIND primary/secondary replication with authenticated zone transfers and change
notifications — not a custom sync protocol. Adding a DNS-only server generates the key and the
matching BIND configuration automatically. Verified against BIND directly, confirming a transfer is
refused without the key and succeeds with it, then again through the real interface.

## Addon domains (Park a Domain)

Account-side self-service: add another domain to an existing hosting account, with its own web
config, home directory, DNS zone, and SSL. Surfaced a gap in account deletion along the way —
removal only cleaned up the primary domain's configuration. Fixed alongside the feature.

## Email Routing, and backfilling older domains

Per-domain control over whether this server or a remote system handles a domain's mail, plus a way
to add SPF/DKIM/DMARC to any domain created before that automation existed — additive, so it
doesn't touch records a customer already has.
