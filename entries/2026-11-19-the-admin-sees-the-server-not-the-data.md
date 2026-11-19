# Phase 313 - The Admin Sees The Server, Not The Data

The three database pages (Maintenance Tools, Database Configuration and Upgrade Database Version) were the last in the
Controller still to be rebuilt, and looking at them with the people who will run this panel in mind turned up a privacy
problem. A hosting company is a processor of its customers' personal data, so its administrators should be able to run the
database server without being able to read what is in it. The old process list showed the text of whatever each
customer's site was asking the database, which can include the names and email addresses of that customer's own customers.

## What changed

Maintenance and Configuration are now one Database Server page. An administrator sees database names, sizes, table counts
and which customer owns each one, and a running or slow query is shown only as its kind and the tables it touches, never
its values. The slow query log, which has to store full query text, now records for a limited time, is locked down and is
erased when it switches off. Checks and optimising run in the background instead of holding the page open, and the
settings page now starts from what the server is really running and applies changes without restarting anything.

Upgrade Database Version now checks the server before it stops anything, and no longer assumes one version of Ubuntu.
Snapshots, which are complete copies of every customer's data, can be deleted and are removed automatically, so they do
not quietly keep personal data for ever.
