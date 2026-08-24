# Phase 127 - Subscribers & Marketing Email, and a Blade Bug Hiding in Plain Sight

The KNJ Network marketing site's "Notify me" form currently posts straight to BillionMail, a
separate hosted tool that's going away as part of the wider web/mail server migration. Before
touching either the form or BillionMail, the panel needed a real replacement built, tested, and
proven live — this phase is that replacement. The web form itself and BillionMail are still
untouched; repointing the one and retiring the other are separate, deliberately-deferred steps.

## What shipped

Account-scoped subscriber lists, each with single or double opt-in and an optional welcome email.
Subscribers can join through a public token-authenticated endpoint, an embeddable form (inline,
card, or modal template, with a generated snippet to drop into any page), or a CSV import — and can
leave through a one-click unsubscribe link. Marketing campaigns can target an existing list, a
manually pasted/typed set of addresses, or a CSV upload of recipients, with open and click tracking
(a 1×1 pixel and rewritten click-redirect links) rolling up into per-campaign stat cards — sent
count, open rate, click rate, bounce rate.

The part worth calling out on its own: **an unsubscribe is honored account-wide, forever, no matter
how a later campaign tries to reach that address again.** A single `subscriber_suppressions` table
is checked once by the send job, right before it builds the actual send batch — not by any
controller at save time, and not per recipient-source. It doesn't matter whether a suppressed
address is sitting in a list, pasted into a textarea, or sitting in a CSV row: the send job resolves
candidates for whichever source the campaign uses, then filters all of them against the same
suppression table before a single row hits `subscriber_campaign_sends`. Verified this holds by
constructing the one case that could slip through — a subscriber whose own `Subscriber` row on a
given list still says `status = 'subscribed'` (simulating having unsubscribed via a *different*
list or a prior campaign, not this one) — and confirming the campaign job excluded them from both a
list-sourced send and a custom-sourced send targeting the same address directly.

The welcome-email and campaign editor is a plain code textarea with a live preview pane and a
merge-tag picker, not a drag-and-drop visual builder. That's a deliberate call, not a scope-cut:
BillionMail's own template editor is a raw textarea with *no* preview at all, which is the exact
thing that made it too awkward to ever actually use for the one welcome email this business wanted
to send. The bar here was "reliably saves, and you can see what you're sending" — a debounced
listener writing into a sandboxed `<iframe srcdoc>` clears that bar with zero new JS dependencies.

## The bug: Blade doesn't know what a string is

Caught this one live on `panel-dev`, not locally — the merge-tag picker's own buttons were showing
their compiled PHP instead of the tag they were supposed to insert: a button that should have read
`{{ subscriber.name }}` instead displayed `<?php echo e(subscriber.name); ?>`, literally, as its
visible label.

The root cause is a genuinely useful thing to know about this templating engine: Blade's `{{ }}`
compiler is a **raw-text regex scan over the whole file**, not something that understands PHP syntax
at all. It doesn't know the difference between a real Blade expression and the literal two
characters `{` and `{` sitting next to each other inside a PHP array literal, inside a comment,
anywhere. The first version of this editor built its merge-tag names as an array of literal
`{{ subscriber.name }}`-style strings for a `@foreach` to loop over — which Blade's compiler happily
"helped" by trying to compile as if they were real echo statements, mangling the file. Rewriting
those strings to build the tag via concatenation inside a real `@php` block
(`'{'.'{ '.$tagName.' }'.'}'`) fixed the buttons — but the file *still* failed to render correctly
after that fix, because a nearby Blade comment explaining the problem also happened to contain a
literal example `{{ }}` pair, and that alone was enough to corrupt compilation past the comment's
own boundaries. The actual fix needed a second pass: no literal `{{`/`}}` sequence anywhere in the
raw file text outside a genuine Blade expression, full stop — comments included. Confirmed the fix
by reading the compiled PHP straight out of `storage/framework/views/` rather than trusting the
rendered page alone, since this is exactly the kind of bug a passing PHPUnit suite won't catch:
nothing in 54 new unit/feature tests exercises character-for-character what a Blade view actually
renders to a browser.

## Verified

Live, end to end, on `panel-dev`, with disposable test data cleaned up afterward down to zero rows
for the test account. A real HTTP POST to the public subscribe endpoint; a real welcome email
confirmed delivered via `doveadm fetch` against the mailbox, not just a queued-job assertion; double
opt-in confirm and one-click unsubscribe both followed through the real links; the suppression edge
case above, confirmed excluded from both a list-sourced and a custom-sourced send; a real campaign
send with the tracking pixel and click-redirect both hit over real HTTP and confirmed bumping
`open_count`/`click_count`; a real CSV import/export round-trip; and the new
`knjpanel:resume-stuck-subscriber-campaigns` sweep confirmed re-dispatching a campaign manually left
in `sending` with stuck `queued` rows — the same "worker got recycled mid-job, nothing should stay
stuck" guarantee this panel already gives every other long-running job.

54 new tests across services, the send job, public controllers, and every account controller,
alongside the existing suite, full green. `pint --test` clean. Shipped in two commits — the feature
itself, then a small follow-up once the Blade bug above was caught and fixed on `panel-dev`.
