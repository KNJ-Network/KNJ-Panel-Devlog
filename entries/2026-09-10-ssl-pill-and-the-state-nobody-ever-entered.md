# Phase 220 - The State Nobody Ever Entered

The panel already had a status pill for exactly this moment: amber, a spinning icon, the label
"Issuing...", built into the same component as every other status pill. It had never been seen doing its
job, not because it was broken but because nothing ever told it to.

## A state that existed and a state that happened

Every certificate status already had a real "in progress" value next to "active" and "failed". The bug
was that a brand-new record started without that state being set, so it showed the value meaning "nothing
has been attempted", the same as a domain untouched for a year. Two different situations wore the same
badge.

## The fix that already existed, right next to the bug

The correct sequence, mark it in progress and then hand it to the background job, was already in place
on two other paths, including the retry button. Two newer entry points had skipped a step that older ones
had learned to include, and nothing enforced it.

## Fixing the second complaint by not inventing anything

The other half of the report was that the page didn't update on its own. A plainer answer already existed
on another page, proven against the same waiting-on-a-background-job situation. Reusing it kept the two
pages predictable for a fraction of the new code, and nothing in the complaint asked for more.

## What was actually missing

Not a status, a color or a refresh mechanism: two lines at the start of two functions, setting the
state that already meant "in progress" before handing off the rest of the work. Everything downstream was
already correct.
