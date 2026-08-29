# Phase 158 - The Nameserver That Couldn't Vouch For Itself

A readiness test on a multi-server test install created an account for the panel's own domain, attached
it to the admin login, and added three mailboxes. Mail worked immediately, but DNS did not: both
nameservers reported "No zones replicating yet," and Main's DNS Zones page showed no zone at all.

The account, site and SSL were fine. Only DNS silently never happened. Zone creation is best-effort,
like SSL and FTP account creation, so a failure is logged and swallowed. The log said only that the zone
file was invalid, because the provisioning script discards the zone checker's output and reports a fixed
string.

Reconstructing the zone content in a rolled-back database transaction and running the zone checker by
hand gave the real answer: the zone's NS records pointed at hostnames inside the same zone, and the
checker refused to load it because those nameservers had no address records.

That is a DNS fundamental rather than a panel bug in the usual sense. When a zone's own NS record points
at a hostname inside that same zone, the zone must also contain matching glue A (or AAAA) records, or
resolving the nameserver would require asking the nameserver, the circular dependency glue exists to
break.

It had never come up because every earlier test domain used nameservers on some other domain. The one
domain where it was bound to break is the panel's own, whose nameservers are subdomains of itself, and
every existing NS test happened to use fixture data that made this path unreachable.

The fix: when building a zone's records, each configured nameserver hostname is checked against the
domain being written, and a matching A (and AAAA, if configured) record is added when it falls inside
the zone. Out-of-zone nameservers get nothing extra. It applies to new-zone creation and to "Restore
Default Records," so zones missing glue are backfilled.

Two new regression tests use domains constructed to be their own nameservers' parent, the shape every
prior test avoided by accident.

Tested (2510/2510).
