# Phase 231 - A Record of the Reader

Removing the two features that let an admin browse a customer's data wasn't the end of the
question, just the easy half of it. A harder version was still sitting there once the obvious
overreach was gone: some access to a customer's private content isn't overreach at all. It's the
job.

## The access that's supposed to exist

Restoring a customer's site from backup needs someone to be able to read that backup. Helping a
customer who's locked out of their own account needs someone to be able to log in as them.
Investigating a delivery problem needs someone to be able to open the message that didn't arrive.
None of that is the kind of thing worth deleting — it's the actual, legitimate work of running a
hosting platform for other people. The difference between this and what was removed earlier
isn't whether the access exists. It's whether anyone could ever tell it happened.

## Trust without a record isn't accountability

A permission check answers one question: can this person do this. It says nothing about whether
they did, when, or to whom. For the handful of actions that reach all the way into a customer's
own content — not settings, not configuration, the actual private material — permission alone
was the only thing standing between "nobody" and "everybody with an admin login," forever, with
no way to look back and ask a simple question later: who read this, and why. That gap doesn't show up as a bug. It shows up as an absence, which is why it took a deliberate review to notice.

## Writing down what a session already knew

Support access is a pair of events: one that starts it and one that ends it. Tying the two together so the whole action can be read as one took a small piece of design.

## A record built to outlive what it's about

A log that disappears the moment the thing it's recording is deleted isn't really a record — it's
a note that erases itself exactly when someone might most want to read it. An admin account that
gets removed, a customer account that gets closed: neither should take the history of what
happened with it. The fix wasn't clever, just deliberate — write down who and what at the moment
it happens, in words that don't depend on either side still existing to be readable later.

## What actually finished

The handful of actions that genuinely need to reach a customer's own content still can. Every one
of them now leaves something behind that says so — not a barrier to doing the work, a record that
it was done.
