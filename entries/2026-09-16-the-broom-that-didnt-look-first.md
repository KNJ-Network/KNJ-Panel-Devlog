# Phase 243 - The Broom That Didn't Look First

A tidy-up step is supposed to be the safe part. Sweep away whatever's left lying around, leave the
real thing behind. Nobody writes a broom that asks first.

## Two different questions wearing the same word

"Remove what shouldn't be here" sounds like one job. It's actually two, and they only look alike
from a distance. The first question is *did we ever agree to this being here* — something was
promised, recorded, committed to, and later that promise was withdrawn. Answering that one is easy
and safe: check the record, and if the record says gone, it's gone. The second question is *is
anything currently sitting in this spot* — no memory involved, no history consulted, just a bare
look at what's physically present right now, with everything present but unrecorded read as
clutter.

The tidy-up step that caused real damage this week was quietly answering the second question while
believing it was answering the first. It looked at a folder, found nothing in its own records about
that folder, and swept it away — never noticing the folder wasn't clutter at all. It was a whole
other, completely unrelated home, one that happened to be sitting physically inside the same larger
space for entirely ordinary reasons, with no relationship to the tidying at all beyond proximity.

## What the safe version actually looks like

The fix wasn't teaching the broom to recognize this one particular case and step around it. That's
the trap — a list of known exceptions is only as good as how complete the list stays, and completeness
degrades the moment anyone adds a new kind of thing that could be sitting nearby.

The real fix was noticing that the *first* question — did we ever agree to this — was already being
answered correctly and safely, one step earlier, by something that had never once gotten it wrong.
That mechanism update anything it had ever agreed to track, and correctly let go of anything it had
agreed to and then un-agreed to. It simply never touched anything it hadn't been told about in the
first place. The tidy-up step was a completely separate, later addition that stopped asking "did we
agree" and started asking "is something here" instead — a strictly broader, strictly riskier
question, bolted on for a narrower reason (clearing out stale leftovers from an earlier version of
itself) than its actual blast radius.

So the fix is subtraction, not addition: stop asking the second question by default. The first
question already does the entire job anyone actually wants done — day to day, that's indistinguishable
from having a perfect list of exceptions, except it never needs updating, because it was never
guessing about what belonged in the first place. Anyone who genuinely wants the broader sweep can
still ask for it explicitly, with a clear warning about exactly what that means — and even then, the
narrow list of known neighbors is still checked first, so an intentional decision to sweep broadly
still can't take out something it should have recognized.

## The other broom, found the same afternoon

A second tool, unrelated on the surface, turned out to share the exact same instinct in a smaller
way: a "find what nobody's watching yet" button that only ever looked in the one spot someone
happened to be pointed at, instead of everywhere it could plausibly check. Not destructive like the
first one — just quietly assuming the person asking already knew where to look, which rather defeats
the point of a tool meant for exactly the case where they don't.

## What actually shipped

A destructive step turned from "always runs, hopes for the best" into "only runs when explicitly
asked for, and even then checks a known-neighbors list first." The safe, already-correct mechanism
sitting one line above it does the everyday job by itself, with nothing extra needed. And a second,
smaller tool stopped assuming where to look and started looking everywhere it reasonably could.
