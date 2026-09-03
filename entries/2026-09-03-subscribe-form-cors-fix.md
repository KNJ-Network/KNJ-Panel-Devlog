# Phase 191 - A Rule That Only Applies to One Kind of Asking

A customer wanted their signup form to feel a little more polished — no jarring page reload when
someone hits subscribe, just a quiet confirmation appearing in place. A reasonable, common request.
Handed to an AI assistant to restyle, it came back looking sharper. It also stopped working
entirely, and nothing about the code itself was wrong.

## Two ways of asking, one of them watched

A browser has two different ways of sending something to another website, and it treats them very
differently. The old-fashioned way — a form that simply navigates to wherever it points, page reload
included — has always been allowed to cross from one site to another freely; that's just how the web
has worked since forms existed. The newer way — a script quietly sending the same information in the
background, then updating the page in place without a reload — is treated far more suspiciously. A
browser won't let a script read the response from a different site's server unless that server
explicitly says it's fine.

The original version of this form used the first, old-fashioned method. The restyled version,
built for that smoother no-reload feel, switched to the second, more modern one — without knowing
that switch came with a new rule attached. Nothing about the visible form was broken. The browser
was simply refusing to let the answer come back, silently, the moment the newer request was made
at all.

## A door that was never told it could be knocked on this way

The receiving side had genuinely never been asked to say "yes, that's fine" to this particular kind
of cross-site knock. It wasn't an oversight in the sense of forgetting — the page had already
described itself as open to being embedded on any external site. It just hadn't accounted for the
one specific way a script-driven version of that embed needs to ask permission, which a plain
navigating form never has to ask for at all.

There's a general switch elsewhere in this system for exactly this kind of permission, already
turned deliberately restrictive everywhere else on purpose — a previous security pass locked it down
hard, because loosening it in the wrong place would have quietly reopened doors that were shut for
good reason. Flipping that general switch back on, even partway, would have undone that work. The
right fix wasn't touching the switch at all. It was giving this one door, specifically, its own
explicit permission slip — leaving every other door exactly as locked as it already was.

## Making sure every kind of answer carries the permission slip

A signup going through cleanly wasn't the only thing that needed to work. Someone mistyping their
own email address needed to see that mistake explained back to them, not the exact same generic
failure a completely broken connection would show. A stale or copied-wrong link needed its own clear
"this isn't a real signup form" response too. Every one of those different answers needed to carry
the same permission slip, or the fix would have solved the easy case and quietly left the harder ones
just as broken. Attaching the permission slip to the whole exchange — not just to the one specific
"it worked" reply — was what made that true for all of them.

## Watching it actually happen, twice

Before writing anything, the failure itself was watched happen live, in a real browser, on the
customer's own actual page — not assumed from the symptom alone. The exact block a browser shows
when this permission is missing appeared right there in the console, naming the missing piece
precisely. After the fix, the same page, the same request, checked again — this time with the
permission present and the block gone.

Tested (2779/2779).
