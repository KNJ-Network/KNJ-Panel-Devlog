# Phase 120 - DNS zone file import/export

The first item off the "genuinely zero" list, smallest first. A zone editor in a comparable panel
lets an admin download a zone as a standard BIND file and upload one to populate a zone. This
closes that gap.

## Export

The easy half: export is a thin wrapper around the same zone-file builder every live write goes
through, so what an admin downloads is what is actually being served, not a separately maintained
representation that could drift.

## Import: parsing a foreign zone file is its own real problem

A new zone-file parser handles a deliberately bounded subset of BIND syntax: single-line
A/AAAA/CNAME/MX/TXT/NS/SRV records, `$TTL`, comments (including inside quoted TXT strings), and the
"blank owner name repeats the previous name" shorthand. It does not attempt full BIND grammar; no
`$INCLUDE`, no `$GENERATE`, no multi-line records. Anything outside the supported shape is reported
as a warning against that line rather than failing the whole import. SOA lines are recognized and
skipped, since the panel generates the SOA itself.

Owner names are relativized against the zone being imported into, and a name fully qualified for a
different domain is rejected with a warning rather than imported into the wrong zone.

## Safety: preview first, additive only

Upload, parse, and show exactly what will be added, with any skipped lines listed, before anything
touches the live zone. Confirming re-parses the same content rather than trusting a submitted
record list.

Import is additive only. A record identical to an existing one is skipped; anything else, including
a same-name record with a different value, becomes an additional record rather than guessing it was
meant to replace the old one. Silently overwriting a working record because the parser guessed
wrong would be far worse than leaving a stale one for the admin to remove. One zone write covers
the whole batch.

## Verified

19 new tests (12 for the parser, 7 feature tests including the reseller-permission boundary); full
suite now 1,729 and the code-style check green. Live-verified on the dev server with a real export
of a test zone.

## Live-verify addendum: round-tripping the export caught a real bug

Feeding the downloaded export straight back into import, the most likely real-world action, gave 12
correctly parsed records plus several confusing warnings. The cause was that the builder emits the
SOA as a standard multi-line block, which the parser did not recognize as a group, so each
continuation line was misread as its own malformed record. The parser now tracks parenthesized
groups, swallowing SOA continuation lines silently and warning once for any other multi-line
record. Shipped as v0.16.51; the same round trip now gives 12 records and zero warnings.
