# Phase 09 - A Real Nameserver, and the Panel's Own DNS Zone Editor

M6 gave the Main server its own authoritative nameserver — BIND9, configured the way real
hosting servers actually run it: it answers for the domains it hosts and refuses everything
else. That second half matters more than it sounds. A DNS server that will recursively resolve
*any* query, for *any* domain, from *anyone* on the internet is the DNS equivalent of an open
mail relay — a well-documented way for a server to get roped into DDoS amplification attacks
without its owner ever knowing. This one won't do that: confirmed live, from an outside IP,
querying a domain this server has nothing to do with, and getting a flat refusal back.

## What it does

Every new hosting account now gets a DNS zone automatically the moment it's created — sensible
defaults set up without being asked: an NS record pointing at this server,
A records for the domain and its `www`, and an MX record so mail (already running on this same
box since the last milestone) actually knows where to go. From there, both the account owner and
the admin side have a full zone editor — add, edit, and remove A, AAAA, CNAME, MX, TXT, NS, and
SRV records, with changes reflected in real DNS answers within the same request.

Zone files aren't hand-patched — every change regenerates the zone file completely from what's
in the database, the same "one source of truth, fully rebuilt" approach already used for this
panel's Nginx and mail configuration. Before any of that ever reaches the live nameserver, it's
checked with the same validation tool BIND itself uses internally — a mistake in one record can't
take the whole nameserver down.

## Two bugs, both caught by actually watching it fail

A serial number BIND uses internally to know a zone has changed was being generated in a format
that silently overflowed the field's size limit — every single zone write failed until that got
noticed and fixed. And the very first attempt at setting a zone's nameserver record pointed at a
made-up hostname *within that zone itself* — which BIND correctly refused, because a nameserver
that lives inside the zone it's serving needs an extra piece of glue data nothing was providing.
Fixed by doing what real hosts actually do: pointing every customer zone at the server's own
already-working hostname, rather than inventing a new one per domain.

Both were found the same way everything else on this build has been verified — running the real
thing against the real server, not assuming a config file was correct because it looked right.

## What's next — and what's staying parked

This milestone deliberately stopped short of DNS-only slave clustering — a second server that
mirrors this one's DNS for redundancy. The mechanism for that (standard DNS zone transfer, not
anything custom) is scoped out and the panel's nav
already has a place reserved for it, but it's explicitly not being built yet. The core panel
comes first; multi-server DNS clustering is a genuine add-on for after that's further along, not
something the rest of the panel needs to wait on.
