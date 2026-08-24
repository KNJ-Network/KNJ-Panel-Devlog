# Phase 127 - Subscribers & Marketing Email, and a Blade Bug Hiding in Plain Sight

The panel needed a built-in subscriber and marketing email feature, built, tested and proven live,
so that account owners can run their own lists without a separate hosted tool.

## What shipped

Account-scoped subscriber lists, each with single or double opt-in and an optional welcome email.
Subscribers can join through a public token-authenticated endpoint, an embeddable form (inline, card
or modal, with a generated snippet), or a CSV import, and can leave through a one-click unsubscribe
link. Campaigns can target a list, a pasted set of addresses, or a CSV upload, with open and click
tracking rolling up into per-campaign stats: sent count, open rate, click rate and bounce rate.

The part worth calling out: **an unsubscribe is honored account-wide.** A single suppression list is
checked once by the send job, right before it builds the batch, rather than by any controller at
save time or per recipient source. Whether a suppressed address sits in a list, a textarea or a CSV
row, it is filtered out before any recipient row is created. We verified this by constructing the
case most likely to slip through: a subscriber who unsubscribed via a different list, and confirmed
exclusion from both a list-sourced and a custom-sourced send.

The editor is a plain code textarea with a live preview pane and a merge-tag picker, not a visual
builder. That is deliberate: the bar was "reliably saves, and you can see what you're sending", which
a sandboxed preview frame clears with no new JS dependencies.

## The bug: Blade doesn't know what a string is

Caught live on the dev server: the merge-tag picker's buttons showed compiled PHP instead of the tag
they should insert. Blade's double-brace compiler is a raw-text scan over the whole file and does not
understand PHP strings or comments. Building the tag strings by concatenation fixed the buttons, but
a nearby comment containing a literal brace pair still corrupted compilation. The fix was to keep
literal double-brace sequences out of the raw file entirely, comments included. We confirmed it by
reading the compiled output rather than trusting the rendered page, since this is the kind of bug a
passing test suite will not catch.

## Verified

Live on the dev server with disposable data cleaned up afterward: a real subscribe request, a
delivered welcome email, double opt-in and unsubscribe through the real links, the suppression case
above, a campaign send with tracking pixel and click redirect counted, a CSV import/export round
trip, and a resume sweep that re-dispatched a campaign left mid-send.

54 new tests across services, the send job and controllers, with the suite green and code-style
clean. Shipped in two commits: the feature, then the follow-up for the Blade bug.
