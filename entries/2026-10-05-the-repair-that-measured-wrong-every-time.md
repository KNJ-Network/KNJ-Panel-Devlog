# Phase 245 - The Repair That Measured Wrong Every Time

Some bugs happen once. This one happened every fifteen minutes, for over a month, and never once
told anyone, because the thing protecting against it was doing exactly its job.

## A self-repair that assumed too much

An older hosting account's webmail and account-area subdomains get upgraded automatically whenever
anything touches that account's configuration again — routing fixes accumulated over months get
applied retroactively, not just baked into new accounts going forward. The mechanism behind that: cut
off the old version of those two blocks, and write a fresh pair in their place.

Cutting cleanly requires knowing exactly where the old version starts. The measurement used a fixed
assumption — count back a fixed number of lines from a known landmark, on the belief that a blank
line always sits in the same place relative to it. True for every configuration this panel writes
for itself. Not necessarily true for one that predates the mechanism doing the measuring.

One real account's configuration didn't have that blank line in the expected spot — an artifact of
how it was first put together, long before this particular self-repair existed to have an opinion
about it. The fixed-distance measurement landed one line short, slicing cleanly through the middle of
the block before it rather than stopping at its edge.

## Why nothing actually broke

Every repair attempt here validates its own output before committing to it, and reverts immediately
if that check fails. The mis-measured cut did fail it, every time — a dangling, unclosed block is not
valid configuration by any definition — so the safety net caught it, restored the original file, and
moved on. The site stayed up, the whole time, correctly. What didn't happen was the actual repair:
the newer routing fixes that should have reached that one account's webmail and account subdomains
never did, because the attempt to deliver them kept failing at the first step, silently, every single
time the surrounding system tried again.

Found by actually reading what had accumulated in the logs rather than assuming a quiet system is a
healthy one — the same failure, for the same one account, logged well over five thousand times since
early September.

## What actually shipped

The fixed-distance assumption is gone. The repair now walks backward from its own landmark until it
finds real content, however much or little blank space happens to separate the two, instead of
betting on a specific amount always being there. Verified against the real, previously-failing
configuration before it ever went near a live server again — the measurement lands in the right
place now, the repair completes, and the account in question finally has the same routing fixes every
newer account already had for free.
