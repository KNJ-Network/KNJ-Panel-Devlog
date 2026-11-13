# Phase 273 - A Licence Server That Raises the Alarm

Every panel checks in with the licence server, so when it has a problem, customers feel it. Until now the
only way to find out was to notice. We gave it a way to tell us.

## What changed

**Checks that run on their own.** The server now looks at itself regularly: whether it is answering, whether
the services and database it depends on are healthy, whether its certificate is close to expiring, whether
disk space or memory is running out, whether updates are being applied and release downloads kept current,
and whether panels are still checking in.

**Alerts that reach us where we are.** Each alert can go to email, Slack, Discord or a custom address, chosen
per alert. A problem alerts once, not every few minutes, and a recovery says so.

**An Alerts page and a banner.** The back office has a page for the alerts, and a banner across the top
whenever a problem is open, so it cannot be missed by whoever opens the console next.

**The sign-in page** now matches the panel's: the dark banner with the card overlapping it.

## Why now

The licence server is the one piece every installation depends on, and it had no one watching it. It now
has.
