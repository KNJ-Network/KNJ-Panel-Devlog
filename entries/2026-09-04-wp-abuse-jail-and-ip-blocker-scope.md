# Phase 200 - The Alarm That Only Covered Its Own Front Door

Noticing unusual CPU use in the Resource Monitor led to a gap in two security features at once, both
traced to the same root cause: they had been built to protect the panel, not the sites it hosts.

## What the monitor was showing

The load turned out to be repeated automated requests against a WordPress endpoint on a hosted site,
a common abuse pattern, and unrelated to any earlier issue. It was handled manually at first, which
also showed that blocking an address doesn't end connections that are already open.

## Two alarms, both wired to the wrong building

Brute-force protection already covered server login and the panel's own login page, but not customer
sites. A bundled generic bot filter turned out to be the wrong tool, since it targets probing for
missing paths, whereas this was abuse of a path that exists. A purpose-built rule was added, tested
against real log samples before use and then by simulating a flood in a test setting.

The panel's IP Blocker also had a gap: it applied blocks only to an account's primary domain, while the
site under attack was a subdomain. The page's own copy claimed a block applied to all of a customer's
sites, which wasn't accurate for subdomains.

## Fixing "which sites" without losing "which address"

A block now has an optional site scope: unset means every site, as before, and a set value means one
domain. The duplicate-prevention constraint was left alone, since one block per address per account,
whichever scope it has, is enough. Rebuilding the web server configuration now computes each site's
list independently, and a site with nothing scoped to it is still rewritten so removed blocks are
actually cleared.

## The door nobody had put a handle on

The IP Blocker had also been owner-only, with no admin-side page, which is why the manual work above was
needed. A new admin page uses the same service and rows as the owner's page, so a block added on either
side shows up on both.

Tested (2852/2852), pint clean, and verified on the development server: the protection bans a
simulated flood, and a block scoped to one site affects only that site.
