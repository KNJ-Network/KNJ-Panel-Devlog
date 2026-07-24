# Phase 09 - A Real Nameserver, and the Panel's Own DNS Zone Editor

M6 gave the Main server its own authoritative nameserver — BIND9, configured the way real hosting
servers run it: it answers for the domains it hosts and refuses everything else. An open resolver
is a well-documented way for a server to be used in DDoS amplification, so this was checked
deliberately: querying a domain the server has nothing to do with, from an outside IP, got a
refusal.

## What it does

Every new hosting account now gets a DNS zone automatically, with sensible defaults: an NS record
pointing at this server, A records for the domain and its `www`, and an MX record so mail knows
where to go. From there, both the account owner and the admin side have a full zone editor — add,
edit, and remove A, AAAA, CNAME, MX, TXT, NS, and SRV records, with changes reflected in real DNS
answers within the same request.

Zone files aren't hand-patched — every change regenerates the zone file from what's in the
database, the same "one source of truth, fully rebuilt" approach used for this panel's Nginx and
mail configuration. Each change is checked with BIND's own validation tool before it reaches the
live nameserver.

## Two bugs, both caught by watching it fail

A zone serial number was generated in a format that overflowed its size limit, so zone writes
failed until it was noticed and fixed. And the first attempt at a zone's nameserver record used a
hostname inside that same zone, which BIND correctly refused without glue data. Fixed by pointing
every customer zone at the server's own working hostname, as real hosts do.

Both were found the way everything else on this build has been verified: running the real thing
against a real server rather than assuming a config file was right because it looked right.
