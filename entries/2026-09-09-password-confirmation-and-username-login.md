# Phase 211 - The Answer Nobody Asked For, Rejecting Itself

Two bugs, found back to back, both live, both in the same neighborhood: a form that hands someone
credentials, and the page that lets them actually use them.

## A password that failed to confirm itself

Auto-generate a password for an account, and the request came back rejected: the password
confirmation didn't match. Nobody had typed a confirmation. Nobody had even seen a password field —
choosing Auto-generate hides the manual entry boxes entirely. And yet there they still were,
sitting in the page, present enough for a browser that remembered a saved login for that exact
account to quietly fill one of the two with something, leaving the other empty, and submit both
along for a ride neither was ever invited on.

Hidden isn't the same as gone. A field that's merely out of sight is still part of the form, still
eligible for a browser's own autofill heuristics, still something the server receives and checks —
and the server's own check for "do these two match" never stopped to ask whether either one was
actually meant to matter for this particular request. The real fix wasn't making the check smarter.
It was making the fields honest about their own relevance: disabled, not just hidden, so a browser
has nothing left to offer an opinion about, and the server's own confirmation check now only ever
runs at all when there's a real manually-typed password in play to confirm in the first place.

## The credentials that only worked half the time

A generated account comes with two things written down: a username and a password. Only one of
them had ever actually been able to get someone through the front door. Hand a real person just the
username — exactly what happens the moment those credentials leave this system and reach someone
else's hands — and the login page simply had no idea what to do with it, because it had only ever
been taught to recognize an email address.

The fix wasn't replacing what the page asks for. It was widening it: check what was typed against
either identifier instead of just one, and stop assuming a login field has to look like an email to
be valid — a plain word gets rejected by a browser's own built-in email-format check before a
server ever sees it, so that assumption had to go too, not just the one further down. Everything
else downstream — how attempts get rate-limited, how a failed one gets logged, which field a
validation error points back to — never had to care which kind of identifier actually came through,
because none of it was ever built around the *shape* of what's typed, only the fact that something
was. Widening the front door didn't require touching a single thing behind it.
