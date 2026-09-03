# Phase 185 - The Tool That Checked Itself

A new safeguard shipped the same day it was designed — a scanner meant to catch a real threat before
it ever reached a live account. It had real tests behind it, a real false-positive check behind it,
and it was still wrong within hours of going out. Not because the idea was bad. Because the one test
that actually mattered — running it against the *whole* real archive it was built from, not the
hand-picked cases used to design it — hadn't happened yet.

## What "should never happen together" actually meant

The rule at the heart of the previous phase's scanner said: if a file mentions a temp directory, reads
something from a request, and either deletes a file or runs one, all in the same file, that's the
shape of the real attack found the night before. True, as far as it went. What it didn't account for
is that "the same file" and "the same few lines" are very different claims once a file gets large
enough. A file with nine thousand lines in it can easily contain all three of those ideas, each for
its own completely ordinary reason, with hundreds of unrelated lines between them. And one specific
file like that turned out to be about as unlucky a false positive as this scanner could possibly have
picked: a piece of WordPress's own core code that every single WordPress site restored through this
feature depends on.

## Finding it before it found someone else

The way this got caught matters as much as the fix itself: not a bug report, not a customer complaint
— running the brand-new tool against the exact real archive that inspired it, deliberately, on
purpose, specifically to make sure it actually held up outside the small set of examples it was
designed and tested against. That's a different, harder bar than "the unit tests pass" — the unit
tests only ever check what someone thought to write a test for.

## The fix is about distance, not presence

The honest correction isn't "check for something extra" — it's changing the question the rule asks in
the first place. Instead of "do all three things appear anywhere in this file," the right question is
"do all three things appear *near each other*." The real attack crams everything into one tight block
on purpose — it's meant to look like a disposable, forgettable snippet, not spread naturally across a
real codebase the way legitimate features are. Checking a narrow window around each mention of a temp
directory, rather than the file as a whole, catches the real shape of the attack while leaving alone
three separate, unrelated ideas that simply both happen to exist somewhere in a big file.

## The same re-run found something else

Running the fixed version back against that same real archive did something unplanned but valuable:
it surfaced three more real backdoors the original manual investigation had missed completely. They
worked identically to the ones already found and removed, with one difference — they read their
instructions from a slightly different place in the request than the others did. A hand search for one
specific spot doesn't find something using a different one; an automated check that covers every
reasonable option does. What started as fixing a false positive ended up correcting the actual count
of what was found in the first place, upward, not down.

## Verifying the fix against both directions of failure

A large, genuinely ordinary file, rebuilt to match the exact shape of the real false positive — left
alone. The real attack's shape, rebuilt with the alternate request source the newly-found backdoors
actually used — still caught. Both directions checked deliberately, not just the one that happened to
go wrong the first time.

Tested (2726/2726).
