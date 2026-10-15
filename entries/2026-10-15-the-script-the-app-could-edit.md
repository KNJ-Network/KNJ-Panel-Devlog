# Phase 264 - The Script the App Could Edit

After the penetration test we audited our own recent work: Deploy from Git's keys, how the panel's paths are
shared with customer websites, the lock on the database tool, and the update process. There was no critical
finding.

## Hardening the privileged parts

One older item was worth fixing on principle: the way some privileged housekeeping was arranged gave the
panel's everyday account more influence than it needed. Defence in depth is meant to stop a small problem
growing into a large one, so that arrangement was tightened, and a test now fails if it ever regresses.

## Signed updates

An update downloaded a release over a secure connection, but nothing proved it was ours. Every release is now
signed, and updates and fresh installs refuse anything that does not verify, or that would roll a server back
to an older release. We tried it on a real machine: a normal update verified, and an unsigned older release
was refused.

## Who is asking

The panel's handling of the visitor address passed along by its web server was tightened so nothing else
running on the machine can vouch for a visitor.

## The small ones

A handful of smaller items were fixed, including how Deploy from Git handles branches and suspended accounts
and how one-time sign-ins are used. One informational item was left alone on purpose and is explained on the
audits page.

Next: more hardening and documentation.
