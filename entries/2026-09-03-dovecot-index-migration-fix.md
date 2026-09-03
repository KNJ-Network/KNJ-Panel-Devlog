# Phase 190 - The File That Came From Somewhere Else

During testing, a mail filter rule ran for the first time after a mailbox had been moved, and folder listing for that mailbox then failed.

## Cause

When a mailbox is moved onto a server, everything in its folder is carried across, including internal bookkeeping files that the mail software builds for itself. One of those, built by a different system, was not in the format this server expects. The filter was the first thing to read it.

Clearing it and letting the software rebuild it fixed the mailbox immediately. The messages and rules were never affected.

## Making sure it does not recur

Both mailbox-moving paths, restore from another host and transfer between servers, now carry messages and user-written filter rules, and leave the software's own bookkeeping behind so each server rebuilds its own.

We confirmed this with a test mailbox holding each kind of file: messages and rules come through, bookkeeping files do not.
