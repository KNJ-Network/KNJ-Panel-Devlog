# Phase 312 - Is Every Report Fresh

Statistics Software was a tick box and a dropdown. It could not say whether the visitor statistics were installed, running or
working, and it told the administrator it enabled statistics "for every account" when the real switch for who could see
them was each hosting package. The server was building reports for accounts that could never open them, and a customer who
asked for an update waited on a page for up to two minutes, which a busy site could not finish.

## What changed

The page now says whether statistics are working, how many sites have a report, when the last update ran and how much disk
it all uses, with a list of what needs attention: a site that failed and why, sites that never got a report, reports that have
gone stale, and packages that leave statistics out. Every site has its own row with a button to update it.

Reports are now built only for packages that include them, in the background, one site at a time and at low priority, so a
large site no longer slows visitors or holds up other work. Customers' update requests are queued instead of holding the
page open. The administrator can set how long to wait for one site and how long old reports and history are kept.
