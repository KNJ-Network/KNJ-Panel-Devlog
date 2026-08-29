# Phase 139 - Real Nameserver Management, and DNS That Actually Follows Mail

Two related pieces of DNS work, both triggered by watching a freshly created Mail Only satellite take
over mail for a domain and asking what a proper hosting-panel admin interface should show at that point.

## A real Nameservers page

Every domain's DNS zone needs NS records, but until now those came from a single legacy nameserver
setting or, if unset, this server's own hostname. That is fine for one box, but does not hold up once an
admin registers ns1 and ns2 and wants every new zone to list both.

A proper nameserver model is now the primary source of truth, with a fallback chain that keeps existing
installs working unchanged: configured nameservers first, then the legacy setting, then the server's
hostname. A new Nameservers page (add, edit, remove, reorder, each with its own IP and optional IPv6)
replaces the old single field on DNS Settings. Adding a nameserver also registers its own A record in
whatever zone matches its parent domain, so it actually resolves once added.

Zone creation now writes one NS record per configured nameserver, and "Restore Default Records" checks
each individually, so a zone with only ns1 present gets ns2 added.

## The bug: MX pointed at the website, not the mail server

For an install with Mail Only satellites, mail on one box and the website on another, switching the
active mail server did not update DNS to match. Every zone's MX record pointed at the bare domain, which
resolves to the website's IP. That works when mail and web share a box, but once split, MX keeps routing
mail to the website's IP.

MX now targets `mail.{domain}`, and that `mail` A record, previously an optional template item, is now
mandatory and tracks the active mail server (the linked Mail Only satellite if one is switched on, this
server otherwise). Every zone is refreshed when an admin switches active mail servers.

Because that refresh only fires on a new switch, a standing "Repair mail DNS records" button on Mail
Settings re-runs it on demand, so an already-configured install can self-heal after updating. It was
live-verified on a multi-server test install already pointed at an external mail server: one click
repaired the stale `mail` record without touching anything else.
