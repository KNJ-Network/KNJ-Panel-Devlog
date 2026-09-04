# Phase 198 - The Animation That Only Ran When Nobody Was Watching

The brief was simple to state and hard to verify: a shield that stays lit, with a glow that rotates
around its edge, colored by how much attention an account needs. It was surprisingly easy to ship
broken, because the one tool available for checking it turned out to be misleading.

## Guessing at a reference, badly, twice

The first draft was built from words alone and rendered as a scribble beside the shield rather than
around it, because a scaling factor added for an orbit ring broke the centering math. The second draft
traced the shield's real outline, which was closer, but only a proper reference showed the intended
effect: the whole outline lit continuously in two colors that trade places as they sweep around, more
like a barber pole than a spark.

## A tool that told the truth about the wrong question

Once the two-tone version was built, it was checked locally by rendering the component in a headless
browser and taking two screenshots a couple of seconds apart. They differed, as expected of something
rotating, so it shipped. In a real browser tab it was frozen.

The automated check wasn't lying; it was answering a different question. Headless rendering runs
pending script to completion before committing a frame, while a normal browser doesn't. The animation
referenced custom properties that a small script set just after render. In a real tab the animation had
already started and locked in whatever those properties held at that instant, and later changes don't
wake it back up.

## Fixing the actual thing, not the test result

A timing patch, such as waiting longer, would still be a race, just a smaller one. The real fix removes
the precondition: the animation is only attached once a stylesheet containing its real keyframe numbers
exists, so there is no moment where it can start before its values do. It was verified in a real tab.

## What's staying unfinished on purpose

The two-tone rotation works, but the polish is left for a later pass. The correct mechanism ships now.

## Everything else this phase finished

A new admin Overview tab provides server-wide stats, a Security Status checklist, a Current Mode panel
and a Recent Findings table. Two sidebar entries collapsed into one tabbed "Malware Scan" parent,
matching the established tab pattern. The account-side owner page, previously reachable only by direct
URL, gained a sidebar entry of its own.

Tested (2842/2842).
