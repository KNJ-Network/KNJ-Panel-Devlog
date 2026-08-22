# Phase 120 - DNS zone file import/export

First item off the "genuinely zero" list, smallest-first as agreed. cPanel's DNS zone editor lets an
admin download a zone as a standard BIND file and upload one to populate a zone — this closes that gap.

## Export

The easy half: `DnsZoneService::exportZoneFile()` is a two-line wrapper around the exact same
`buildZoneFile()` every live write already goes through, so what an admin downloads is byte-for-byte
what BIND is actually serving — not a separately-maintained representation that could drift from it.

## Import — parsing a foreign zone file is its own real problem

`CpanelImportService` (built weeks ago) explicitly documents that it deliberately avoids parsing the
zone file inside a cPanel backup, because "parsing the archive's own BIND zone syntax is its own
unverified-format problem, not attempted here." That's exactly right, and exactly what this feature
had to actually solve rather than defer again.

New `ZoneFileParser` handles a deliberately bounded subset of real BIND zone-file syntax: single-line
A/AAAA/CNAME/MX/TXT/NS/SRV records, `$TTL`, comments (including inside quoted TXT strings), and the
common "blank owner name repeats the previous line's name" shorthand. It does **not** attempt full BIND
grammar — no `$INCLUDE`, no `$GENERATE`, no parenthesized multi-line records. Real-world exported zone
files (from another host, or `dig axfr`) are overwhelmingly one record per line in this shape; anything
outside it is reported as a warning against that specific line rather than failing the whole import.
SOA lines are recognized and silently skipped (this panel synthesizes SOA fresh from settings on every
write, same as it always has — it's never stored per-record).

Owner names get relativized against the zone being imported into (`www.example.com.` → `www`,
`example.com.` → `@`) — a name fully-qualified for a *different* domain is rejected with a warning
rather than silently imported into the wrong zone.

## Safety: preview first, additive only

Upload → parse → show exactly what will be added, with any skipped lines listed, before anything
touches the live zone. Confirming re-parses the same content rather than trusting a submitted record
list, so what gets applied is always derived straight from the zone-file text.

Import is additive only, matching Restore Default Records' own posture: a parsed record identical to
one already in the zone (same type, name, and value) is skipped; anything else — including a same-name
record with a *different* value, e.g. importing a new IP for an A record that already exists — becomes
an additional record rather than guessing it's meant to replace the old one. Silently overwriting a
working record because the parser guessed wrong would be far worse than leaving a stale one for the
admin to remove by hand via the existing zone editor. One zone-file write covers the whole import
batch, matching "Save All Records"' own "one regeneration, not one per row."

## Verified

19 new tests (12 for `ZoneFileParser` covering the record types, blank-name inheritance, unsupported
types/directives, invalid values, TTL clamping, comment stripping; 7 feature tests for export/import
including the reseller-permission boundary) — full suite now 1,729 (up from 1,710), `pint --test` green.
