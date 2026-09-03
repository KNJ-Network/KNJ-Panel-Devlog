# Phase 192 - The Same Symptom, a Different Door

A fix went out. The customer updated their server. They tried the form again, on camera with
someone watching over their shoulder. It still didn't work. Same message, same console error,
same everything — as far as anyone could tell from the outside, nothing had changed at all.

## Trusting the symptom too far

It's tempting, the second time a bug shows the exact same face, to assume it's the same bug —
that the fix simply didn't take, or missed some case. That assumption is worth resisting. A
symptom is just what a system chooses to show on the outside; several unrelated causes can
produce the identical-looking failure, especially when the failure is as blunt as "the browser
refused to let this through." The right response to "it's still broken" isn't "the fix must be
wrong" — it's "watch it happen again, from scratch, and see if it's actually the same thing."

Watched live, a small but decisive difference turned up. A plain, direct request to the real
address — bypassing the browser, bypassing the form, bypassing everything upstream — came back
labeled "not found." Not blocked. Not refused permission. Simply absent, as if the address itself
didn't exist. That is not what a permission problem looks like from the inside, even though it can
look identical from the outside, in a browser console that only knows how to report one kind of
failure for both.

## A shortcut with an old map

The real address did exist — checked directly against the record that was supposed to answer to
it, and it was there, valid, correctly configured. So something between the request and that
record was quietly redirecting it somewhere else. And it was: a convenience shortcut, built so a
domain's own address could double as a doorway straight into its panel, had a fixed idea of what
"account area" traffic looks like, and it had never been told that this particular kind of request
— a public signup form, meant to be reachable by absolutely anyone, no login involved — wasn't part
of that at all. It got swept into the same "belongs to the account area" pile and rerouted
accordingly, straight into a corridor that led nowhere. From the outside: identical symptom, gone
address, generic browser complaint. From the inside: a signpost pointing the wrong way, on a
route the earlier fix had never had a reason to look at.

This wasn't a brand-new mistake, either — it was the same shape of oversight the login page itself
had hit once already: a shortcut built to prefix nearly everything, minus a short, explicit list of
exceptions carved out one at a time as they were found. That earlier list already knew to leave
login, logout, and a handful of other top-level pages alone. It had simply never been told about
this newer, entirely different kind of top-level page — one that didn't exist yet the last time
that list was written.

## Fixing the map so it doesn't need to be told twice

The fix itself was small: add the missing kind of address to the same exception list the login
page already lives on. The more useful part was making sure every domain that had already been set
up before this fix existed would pick it up on its own — the same self-repairing approach already
used for the login-route fix, extended with one more marker to watch for, so the next time
anything touches a domain's configuration at all, it quietly checks itself against the current
rules and catches up if it's behind, with no one having to remember which domains need attention.

A second, smaller piece of insurance went in alongside it: the code responsible for attaching the
one permission header a cross-site signup request actually needs was rewritten to be explicit about
surviving an error response, not just a successful one — not because a gap was found, but because
relying on an internal detail of how the framework happens to behave today, without saying so out
loud, is the kind of thing that quietly stops being true in some future version. Written down and
guarded against now, cheaply, rather than rediscovered the hard way later.

## Two failures, one console message

The lasting lesson isn't really about nginx rewrites or CORS headers specifically. It's that
"it's still broken, same error" is a report about what the person on the other end can see, not a
diagnosis of what's actually wrong underneath it. Watching the failure happen again, all the way
down to a request that never even reached the code meant to handle it, was the only way to tell
these two problems apart — they'd have looked exactly the same from a screenshot alone.

Tested (2780/2780).
