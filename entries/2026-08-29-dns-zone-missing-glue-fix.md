# Phase 158 - The Nameserver That Couldn't Vouch For Itself

Did a live readiness test on the real 4-server production stack: created an account for the panel's
own domain, attached it to the admin login, added three mailboxes. Mail worked immediately — the
Mail Only satellite's dashboard showed all three. DNS didn't. Both nameservers reported "No zones
replicating yet," and Main's own DNS Zones page said the same thing: no zone at all, for a domain
that had supposedly just been provisioned.

The account itself was fine. The site was fine. SSL had actually issued. Only DNS silently never
happened. `ProvisionAccountJob` treats zone creation as best-effort — same as SSL, same as FTP account
creation — so a failure there gets logged and swallowed, not surfaced anywhere an admin would see it
without going looking. The log line: `Failed to write DNS zone: ERROR: invalid zone file for
knj.network`. Not helpful on its own — the provisioning script pipes `named-checkzone`'s real output
to `/dev/null` and reports a fixed string on failure, so the actual reason never reaches the app.

Reconstructed the exact zone content the service would have generated (via a rolled-back DB
transaction and some reflection to call the private builder methods directly) and ran
`named-checkzone` against it by hand:

    zone knj.network/IN: NS 'ns1.knj.network' has no address records (A or AAAA)
    zone knj.network/IN: NS 'ns2.knj.network' has no address records (A or AAAA)
    zone knj.network/IN: not loaded due to errors.

That's the real answer, and it's a genuine DNS fundamental, not a panel bug in the usual sense: when a
zone's own NS record points at a hostname that lives *inside that same zone* — a nameserver that's a
subdomain of the domain it serves — BIND refuses to load the zone at all unless a matching glue A (or
AAAA) record for that nameserver also exists in the zone. Without it, resolving the nameserver's own
address would require asking the nameserver, which is exactly the circular dependency glue records
exist to break.

This had never come up before because every previous test domain — `example.com`, `multi-ns-test.com`,
every disposable `.test` domain this whole readiness pass created — used nameservers on some *other*
domain (`ns1.knj.network`, unrelated to the site being tested). The one domain where it broke is the
one domain where it was always going to break: the panel's own, whose nameservers are literally
`ns1.`/`ns2.{itself}`. Every existing NS-record test in the suite happened to pick fixture data that
made this exact path unreachable.

Fix: when building a zone's record set, check each configured nameserver's hostname against the
domain being written. If it falls inside that zone — a suffix match, or an exact match for the apex
itself — add a matching A record (and AAAA, if one's configured) for it, right alongside its NS
record. Out-of-zone nameservers get nothing extra, same as before; they don't need glue, because
whoever's authoritative for *their* domain already vouches for their address. Wired into both
`createZoneForSite()` (new zones) and `restoreDefaultRecords()` (so an account that's missing glue for
some other reason gets it backfilled the same additive way everything else on that page works).

Two new regression tests, both against domains constructed to actually be their own nameservers'
parent — the shape every prior test avoided by accident. One drives `createZoneForSite()` straight
through and checks the glue lands; the other seeds a zone with the NS record already present but the
glue missing and checks `restoreDefaultRecords()` backfills it.

Tested (2510/2510).
