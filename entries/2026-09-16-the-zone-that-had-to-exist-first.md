# Phase 242 - The Zone That Had To Exist First

Handing a domain over sounded like the last step. It kept turning out to be the second-to-last one.

## What a promise to answer actually requires

A domain delegated to a nameserver is a promise: ask me and I'll tell you where things live. Asking
for a certificate tests that promise harder than anything else does — it doesn't just check the one
address being certified, it walks the whole family tree above it, asking the same question of every
ancestor along the way. A promise that only covers the one address it was made for isn't the promise
a certificate authority is actually listening for.

That gap had already been patched once, quickly, the same day it was found — a small, self-contained
answer conjured up just for the one address that needed it, live on a real spare machine, working
exactly as far as the immediate question and no further. It held up fine for what it was asked. It
was still the wrong shape of fix, and worth saying plainly why: a patch invented to answer a question
on someone else's behalf needs its own memory of having done so, and its own discipline about
stepping aside the moment something real takes its place. That's not a smaller job than the real
fix — it's a different, harder one, and building it carefully wasn't worth doing just to avoid
building the real thing. Reverted the same week it was tested, once the real shape was clear.

## The real shape

Don't answer on the family's behalf. Make the family real first.

A single account, created for real, produces a real zone as a side effect — nobody has to invent
anything, because a genuine hosting account was always going to need genuine DNS. The fix was never
about the certificate at all. It was about which order two already-planned things happen in: don't
ask the hard question until the account that would have answered it correctly already exists.

So the two things that used to be separate — set up your first domain, and create your first hosting
account — became one guided flow instead of two. A package, chosen or auto-filled. A domain and an
owner, the exact same choice already offered anywhere else an account gets created. A hostname,
typed as just the part that's actually new, with the rest supplied automatically from what was
already entered a screen earlier. And, before any of it commits to anything: a real check that the
domain's own nameservers already point here — not a soft warning, an actual stop, because everything
past this point is a real account and a real zone, not a certificate attempt that costs nothing to
retry.

Keeping your own DNS provider stays exactly what it always was — a first-class choice, not a
consolation prize for people who didn't take the recommended path. Nothing about accounts or packages
belongs in that story, so nothing about it changed.

## The thing that was quietly waiting to be asked correctly

Elsewhere, cleanly unrelated, a much smaller promise was breaking in a much smaller way: asking to
remove something installed at the very top of a folder instead of one level inside it. The code that
actually does the removing already knew exactly how to handle a top-level install correctly — it had
been taught that months ago. The code that decides whether a request is even allowed to reach that
point hadn't caught up, checking for a narrower shape than the one it was actually being asked to
handle. Every top-level install, of every kind this panel knows how to install, was hitting the same
wall before ever reaching the part that already knew what to do.

## What actually shipped

One flow instead of two, in the order that finally makes the hard question answerable the first time
it's asked. The earlier same-day patch is gone, not superseded-and-forgotten-about but actually
removed, its job done properly by something that doesn't need remembering to clean up after itself.
And a narrower, older gap — asking to remove the wrong shape of thing — closed the same week it was
found, for good, for every kind of thing this panel knows how to install.
