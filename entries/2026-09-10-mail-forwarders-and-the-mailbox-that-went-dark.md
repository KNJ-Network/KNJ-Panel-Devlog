# Phase 219 - The Mailbox That Went Dark

An address can have two things pointed at it: a real inbox, and a rule that says "also send
this somewhere else." Nothing about those two ideas actually conflicts. But the system
underneath them only had one lever, and pulling it did not add — it replaced.

## A feature with a silent opposite

Creating a forwarder for an address that already had a mailbox didn't fail. It didn't warn.
It succeeded completely, updated exactly the row it was supposed to, and from that moment on
the mailbox stopped receiving anything, forever, with nothing in the panel ever saying so.
The forwarder wasn't malfunctioning — it was doing precisely what a forwarder does, at the
one layer of the mail system where "redirect this" and "replace this" happen to be the exact
same instruction. The two features never fought each other. They just could not both be true
at once, and nothing was built to notice when someone asked for both.

## Two ways to add "and", only one of them safe

The obvious-looking fix is to make one rule aim at two destinations instead of one. It's the
kind of change that sounds like a formatting detail — is a redirect a single address, or a
list — until you look at what has to answer it. The address it would be added next to isn't a
plain destination, it's a rewrite target: a redirect, once matched, can be looked up again by
what it just became, in a system already handling that expansion internally. Add a second
destination there and the failure mode isn't an error message, it's a live mail server
behaving in some unverified way at the exact layer that decides whether a real customer's
email is delivered anywhere at all. Two lines to write, in the one file where writing the
wrong two lines gets found out in production, on a system that has to stay correct all the
time, for everyone, not just for the one address someone was testing.

The other way to add "and" is to not touch that layer at all. Deliver the mail once, normally,
the way it already arrives — and let the mailbox's own local rules make a copy on the way in,
the same mechanism that already knows how to file a message into a folder or reply to it
automatically. The rewrite-and-replace machinery never even sees the request; it's told this
address isn't a redirect anymore, full stop. Everything about "also forward this" happens one
layer downstream, in code that only ever affects the one mailbox it's attached to.

## What refusing to guess actually cost

Slower. The second path meant reading how a message already gets filed once it lands — not
assuming, reading — and extending that same mechanism rather than reaching for the shortcut
that looked like it would work in one line at the layer with the most to lose. The extra time
went into confirming that the delivery mechanism already used here has an explicit "keep going
after this" mode, distinct from the version that would have caused the exact same silent
replacement this whole feature exists to fix, from a different starting point. The right
version does one small, boring thing — it says keep the original *and* send this — from
inside the one place already scoped to a single mailbox, where getting it wrong breaks exactly
one thing instead of a shared system nobody was asked to gamble with.

## What a checkbox is standing in for

Every screen shows a control smaller than the decision behind it. What looks like a checkbox
for "also keep a copy" is a stand-in for two separate refusals holding the door open: a
forwarder can no longer be created at an address that already quietly had a mailbox, and a
mailbox can no longer be created somewhere a forwarder was already silently winning. Both of
those used to succeed instantly and wrongly. Now they ask, once, at the one moment asking
still means something — before anything has gone dark, not after.
