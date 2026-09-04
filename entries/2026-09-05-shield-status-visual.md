# Phase 198 - The Animation That Only Ran When Nobody Was Watching

The brief was simple to state and hard to actually verify: a shield that stays lit, with a glow
that rotates around its own edge, colored by how worried anyone should currently be about a given
account. Simple to describe. Surprisingly easy to ship broken without ever noticing, because the
one tool available for checking it turned out to lie.

## Guessing at a reference, badly, twice

The first draft was a guess — no reference image, just the words "embossed outline" and "glowing
orb with a trail" turned directly into SVG. It rendered as a red scribble sitting near the shield,
not around it, because a scaling factor introduced to leave room for an orbit ring quietly broke
the centering math that placed the shield in the first place. The second draft fixed the
centering and traced the shield's actual outline instead of a separate ring — closer, but still
wrong in a way that only became clear once an actual reference arrived: a stock "neon shield"
clip, extracted frame by frame with `ffmpeg` since there was no way to play the video directly.
The real thing wasn't a faint comet chasing a mostly-dark edge at all. It was the whole outline,
lit continuously, split into two colors that traded places as they swept around together — closer
to a rotating barber pole than a spark.

## A tool that told the truth about the wrong question

Once the two-tone version was built, the obvious next step was checking it actually moved. No way
to take a live screenshot of the running panel existed in this setup, so the check ran locally
instead: render the component's real output, drop it in a headless Chrome instance, and take two
screenshots a couple of seconds apart. They came back different — light and dark had swapped
sides, exactly as expected of something rotating. Confident, that got shipped.

It was frozen. Completely static, confirmed directly in a real browser tab, no ambiguity about it.

The honest answer is the automated check wasn't lying about what it saw — it was answering a
question that didn't match how a real browser behaves. Headless Chrome's deterministic rendering
mode runs pending script to completion before it ever commits a frame; a normal browser doesn't
make that guarantee. The animation's CSS `@keyframes` pointed at custom properties that a small
script set immediately after the shield rendered — a completely ordinary pattern — but the
animation itself was already running by the time that script's assignment landed, and the browser
had already locked in whatever those properties held at that instant: nothing, or the browser's
own default. Once locked, later changes to the same properties don't retroactively wake the
animation back up. In the deterministic test environment, the script finished before any of that
mattered. In a real tab, it didn't.

## Fixing the actual thing, not the test result

The fix wasn't a timing patch — waiting longer, deferring the script further — because any race
condition solved by "wait a bit" is still a race condition, just a smaller one. The real fix was
removing the race's precondition entirely: the animation itself is never attached to an element
until a stylesheet containing its real, already-known keyframe numbers exists. No custom
properties read at some ambiguous later point, no window where the browser could commit to a
stale target, because there's no longer a moment where the animation exists before its numbers
do. Verified the only way this specific bug could be — the same person who caught it live,
looking at the same real tab, confirming it moved this time.

## What's staying unfinished on purpose

The two-tone rotation now genuinely works, on the actual reference it was built against, but it's
not the final version — the user's own words, mid-fix: "we can play with this later on down the
line and maybe make a custom animation file for it." That's a real, deliberate choice to ship the
correct mechanism now and revisit the polish separately, not a corner cut. Recorded here so it
doesn't quietly disappear from the list.

## Everything else this phase actually finished

A new Controller-side Overview tab exists where nothing did before — server-wide stats, a Security
Status checklist, a Current Mode panel, and a Recent Findings table, all backed by two new
`MalwareScanService` methods (`accountStats()`/`serverStats()`) pulling from real scan and finding
rows. Two separate sidebar entries collapsed into one tabbed "Malware Scan" parent, matching the
Subscribers feature's own established tab pattern. The account-side owner page — previously
unreachable except by direct URL or a forced suspension redirect, with no sidebar entry of its own
at all — finally got one.

Tested (2842/2842).
