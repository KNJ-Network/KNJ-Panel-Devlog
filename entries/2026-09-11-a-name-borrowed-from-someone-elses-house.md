# Phase 228 - A Name Borrowed From Someone Else's House

A subdomain in this panel has never owned its own patch of DNS. It lives inside its parent's zone and
isn't delegated anywhere. That has a consequence: a feature that writes "this domain's" records has to
be told whether "this domain" means the zone it is editing or just a name inside it.

## The check that only knew how to check itself

A subdomain's SPF and DKIM records had a gap: nothing had ever offered to write any. The repair button
that exists for other domains didn't appear for a subdomain at all. It wasn't broken, just never built
for a domain with no zone of its own. Subdomains can send mail, so the gap wasn't theoretical.

## Two protocols, two different rules

DMARC turned out to be a separate question. SPF and DKIM are strict: to send as a name, that name needs
its own record. DMARC was designed with an escape hatch, where a name with no policy inherits its
parent's. A subdomain without its own DMARC record is covered, just not under its own name. Treating
both the same would have been wrong for one of them.

## Writing into a zone that isn't yours

Writing a subdomain's records means editing the parent's zone at an interior name. The mail-record
writer had only ever written at the top of the zone. It now writes at an interior name and generates a
signing key that belongs to that name, never borrowed from the parent.

## What actually finished

A subdomain that sends its own mail can now get its own SPF and DKIM coverage, and one that doesn't gets
an accurate DMARC answer instead of a false alarm.
