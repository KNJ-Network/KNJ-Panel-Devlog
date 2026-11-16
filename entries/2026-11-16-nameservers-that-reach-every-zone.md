# Phase 282 - Nameservers That Reach Every Zone

A nameserver list looks like a small thing: a couple of names and addresses. It turned out to hide the most surprising
behaviour in the DNS pages. The list was written into a domain's DNS only when the domain was first set up, so adding,
renaming or removing a nameserver later quietly did nothing for every domain that already existed. Nothing said so.

## What changed

**Existing domains follow the list.** When some domains no longer match, a banner says how many, and an update page
shows exactly what would change in each before anything does: a nameserver that is missing, an address a nameserver
inside its own domain needs, and which nameserver is named as the primary. Removing a nameserver from domains that still
list it is a separate choice, never the default, because it can take a domain offline if its registrar still points at
that nameserver. Every change is kept in the domain's history and can be undone there, and a domain that cannot be
updated is left exactly as it was without stopping the others.

**Each nameserver says whether it works.** Its name resolves to the address it should, as the rest of the internet
sees it; it answers DNS questions there; the address belongs to this server or a linked DNS-only one; and the page says
how many domains use it. It also warns about a single nameserver, or two that share an address and so protect nothing.

**Changing the order says what it does.** The first nameserver is the primary, and the page now says which one that is
and when a change reaches each domain.

**A first-run setup that explains the part the panel cannot do.** Two names with this server's addresses suggested, then
what to register at the domain's registrar and where to point each domain, in order.
