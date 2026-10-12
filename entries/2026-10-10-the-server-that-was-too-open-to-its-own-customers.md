# Phase 254 - The Server That Was Too Open to Its Own Customers

The last two audits looked at the server itself, from the point of view of one hosting customer curious about
their neighbours.

## Readable by default

Some things a shared server writes for its own use were more widely visible between accounts than they
needed to be. None was a break-in on its own; together they made it easier than it should be to learn about
other people's sites. These are now closed down, new installs get this at setup, existing servers get it on
upgrade, and a test pins it so it cannot quietly regress.

## Leftovers

Removing an account used to leave some things behind. They are now removed with the account. Smaller
hardening items rounded the list out. The full write-up is on the Security Audits page.
