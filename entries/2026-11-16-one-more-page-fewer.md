# Phase 288 - One More Page Fewer

Perform a DNS Cleanup compared the zones the nameserver was serving with the zones the panel believed it hosted, and showed
the two ways they can drift apart: a zone served that no account owns, and a zone a domain depends on that is not served at
all. Both are questions about the health of a zone, which is what the DNS Zones list now answers for everything else, so a
separate page you had to remember to open made little sense.

## What changed

The page is gone and its two checks moved into the zone list. A hosted zone the nameserver is not serving is marked in the
list and on its own page, with a button that rewrites it from the panel's records. Zones nobody owns appear as a banner,
and each opens a review before anything is removed. The review says what the nameserver answers for the zone and, most
importantly, whether the internet still sends the domain's queries to this server, because removing a zone for a domain
that still points here would take it offline. In that case removal needs an extra confirmation.

The comparison now runs in the background every few minutes instead of when the page is opened, and a zone that stops
being served is reported to administrators, once, because that is a domain whose website and mail may have stopped working
and nothing used to say so until somebody happened to look. Repairs and removals are now written to the action log.

## A fix worth knowing about

If the nameserver could not be asked which zones it served, the old page treated that as "nothing is wrong": it reported
that everything agreed and every zone was live. A failed check now says that it failed, and why, and shows nothing as fine.
