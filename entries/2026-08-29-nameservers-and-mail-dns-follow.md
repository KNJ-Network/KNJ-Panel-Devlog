# Phase 139 - Real Nameserver Management, and DNS That Actually Follows Mail

Two related pieces of DNS work today, both triggered by the same real-world moment: watching a
freshly-created Mail Only satellite actually take over mail for a domain, and asking what a proper
hosting-panel admin interface would show at that point.

## A real Nameservers page

Every domain's DNS zone has always needed an NS record pointing somewhere, but until now that
"somewhere" was a single legacy `dns.nameserver_hostname` setting, or — if that was never set — this
server's own hostname. That's fine for a one-box install, but doesn't hold up once an admin actually
registers ns1/ns2 and wants every new zone to list both, the way established hosting-panel
conventions do.

Built a proper `DnsNameserver` model as the primary source of truth, with a fallback chain that
keeps every existing install's zones working unchanged: configured nameserver rows first, then the
legacy setting, then the server's own hostname. A new Nameservers page (add/edit/remove, reorder,
each with its own IP and optional IPv6) replaces the old single hostname field on DNS Settings.
Adding a nameserver also does something a lot of panels skip — it registers that nameserver's own A
record in whatever zone matches its parent domain, so `ns1.example.com` actually resolves once it's
added, not just referenced.

`createZoneRecords()` now writes one NS record per configured nameserver instead of a single
hardcoded line, and the self-service "Restore Default Records" recovery action checks each
configured nameserver individually — otherwise a zone with only ns1's record already present would
never get ns2 added, even though ns2 is genuinely missing.

## The bug: MX pointed at the website, not the mail server

While building the above, a real question came up: for an install running Mail Only satellites —
mail on one box, the website on another — does switching the active mail server actually update
DNS to match? It didn't.

Every zone's MX record was written as the bare domain (`example.com`), which resolves through the
zone's own apex `@` A record. That record has to always point at the website's IP, since it's also
what every visitor's browser resolves. For a normal single-server install this happens to work,
because mail and web run on the same box. The moment they're split — exactly the Mail Only
architecture this panel builds toward — MX silently keeps routing incoming mail to whichever server
answers on port 25 at the website's IP, which might be nothing at all.

Fixed the actual coupling: MX now targets `mail.{domain}` instead of the bare domain, and that
`mail` A record — previously an optional, togglable zone-template item — is now mandatory and tracks
whichever mail server is actually active (the linked Mail Only satellite if one is switched on in
Mail Settings, this server's own IP otherwise). Every zone on the server gets refreshed
automatically the moment an admin switches active mail servers, via a new
`DnsZoneService::refreshMailRecords()` call wired into the switch action.

That refresh only fires on a *new* switch, though — an install that already switched to an external
mail server before this fix shipped would never trigger it. Added a standing "Repair mail DNS
records" button on Mail Settings for exactly that case: re-runs the same refresh on demand,
independent of switching, so an already-configured install can self-heal after updating the panel.
Live-verified against a real 4-server stack already pointed at an external mail server — one click
correctly repaired the stale `mail` A record without touching anything else.
