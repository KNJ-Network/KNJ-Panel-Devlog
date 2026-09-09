# Phase 207 - The Server That Wasn't In Its Own Table

Two update-related controls for this server lived in two different places. One card listed every
other server and let each be checked, updated and rebooted from one screen. A second, separate card
handled updates for the server being looked at, with its own buttons and layout. The obvious question
was why this server wasn't just a row in the same table.

## Same row, different plumbing underneath

Folding the local server into the table didn't need a new kind of row. Pending count, reboot-needed and
status each already had a "read from a remote server" version and now needed a "read from here" version
next to it. Update and Reboot already existed for this server; the table just points at the right one
for each row.

The code that watches for a run to finish and updates a row's status already matched rows by their own
identifier, so giving the local row the same identifier lets the same code update it too.

## A card that used to always talk gets asked instead

The old package table was fully populated on every page load, though nobody looks at more than one
server at a time. It is now empty by default and fills in when you choose a server, with no page
reload. For remote servers, the routine check now returns the package list along with the count, so it
is cached and ready on request.

## Two things caught before they shipped

One was a genuine mistake: the card's heading didn't update on an error, so a message for one server
could sit under a stale title naming another. The error path now updates what the success path does.
The other looked like a bug and wasn't: a server that hadn't yet been brought up to the same release
didn't send the extra detail, so it showed a pending count with an empty list until it caught up.
