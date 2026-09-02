# Phase 176 - The Subdomain That Never Got Its Address

A restored account had four subdomains. Three of them didn't resolve. The obvious suspects were all
innocent — the vhosts existed, nginx was configured correctly, DNS itself was healthy. The zone file
just didn't have the A records. Only the first subdomain's.

## A property access is not always a fresh read

`DnsZoneService::buildZoneFile()` reads a zone's records to write out the actual BIND-format zone
file, via `$zone->records` — Eloquent's own convenient shorthand for a relationship, which loads once
on first access and then quietly answers every later read from that same cached copy. That's normally
exactly the right behavior; re-querying the database every time a page touches `$model->relation`
would be wasteful. It stops being right the moment something else creates a new related row on that
same model object in between two reads.

`SubdomainService::create()` does exactly that, once per subdomain, reusing the same parent `Site`/
`DnsZone` object across the whole loop when a restore recreates several subdomains at once. The first
subdomain's own A record gets added to the zone correctly — at that point the cached collection is
either empty or freshly loaded, so it's still accurate. Every subdomain after the first calls
`buildZoneFile()` against a `$zone->records` that was cached back before any of this loop's own writes
happened. The record for subdomain #2 gets written to the database. The zone file BIND actually serves
does not include it, because the code building that file never saw it exist.

## The fix is the method call, not the property

`$zone->records()->get()` — records() with parentheses, not the bare property — issues a real query
every time, current as of the moment it's called. Same data most of the time, genuinely different data
in exactly the scenario that broke here. One-line fix, in the one function that was actually wrong;
see the companion entry on the malware scanner and the import-log duplication fix for two more spots
this same trap showed up in in this codebase, one real, one worth guarding against on principle.

## Verifying it against something that actually resolves

Live on panel-dev: restored an account with multiple subdomains, confirmed via direct DNS lookups that
every single one — not just the first — now resolves to a real A record. A regression test reproduces
the exact shape of the bug directly: caches the relation on a zone before adding a new record through
it, confirms the OLD code path drops the new record from a rebuilt zone file, confirms the fix doesn't.

Tested.
