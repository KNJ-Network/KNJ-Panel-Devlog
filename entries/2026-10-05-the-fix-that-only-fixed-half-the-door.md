# Phase 244 - The Fix That Only Fixed Half the Door

A bug that was already fixed once has a particular way of being annoying the second time. Nobody's
looking for it. The fix is already filed away as done.

## Two doors, one lock

A daily maintenance job — nothing glamorous, just clearing out messages already marked deleted —
had been failing on mail servers. Not loudly. A background
job failing quietly is the worst kind: nothing visibly breaks, so nothing prompts anyone to look.

This exact failure had already been diagnosed and fixed once before, back in September — a scheduled job that was set up incorrectly and failed every time. The fix at the time corrected its configuration.

It was a real fix. It was also, it turned out, a fix for only one of two doors into the same room.

## The second door nobody knew was there

The thing actually responsible for running that cleanup job had quietly changed underneath the
original fix. A software update to the mail client itself had introduced its own, separate
scheduling mechanism for the exact same job — and on a modern system, that new mechanism is the one
that actually runs; the old one steps aside and does nothing the moment it detects the newer system
is present. The September fix corrected a door that had already stopped being the entrance.

Nobody could have caught this by re-reading the original fix harder. The fix was correct for the
world it was written in. The world underneath it changed later, quietly, the way a routine software
update changes things — and the thing it changed was something the original fix had reasonable cause
to assume would stay put.

## What actually shipped

The real, currently-active mechanism got the same treatment, but this time written as a layer on top of the vendor's own configuration rather than a full replacement of it, specifically so a future software update can
keep updating its half without silently erasing the fix again. Applied to every server that runs
this job, confirmed working end to end, not just deployed and assumed.
