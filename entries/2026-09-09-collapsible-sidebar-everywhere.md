# Phase 209 - The Two Groups That Got To Fold Up

Two sections of the sidebar could fold up out of the way. Every other section, in both the admin
panel and the account panel, was stuck open forever — a plain list, take it or leave it, no matter
how far down the page it pushed everything else.

## The exception had already done all the hard thinking

Building the first collapsible group had meant working out several real questions at once: what
"collapsed by default" should actually mean, how to keep whichever section holds the current page
from ever hiding it, how to remember a choice once someone deliberately makes one, how to behave
correctly when a live search is also filtering the list underneath. All of that already existed,
proven, for exactly two groups. Extending it to every other one wasn't a second design — it was
recognizing that the condition separating "gets this treatment" from "doesn't" was never really
about which section it was. It was one default value, quietly set the conservative way the first
time around. Turning it around took one line, not a rewrite.

## One button standing in for the whole list

The new expand-all/collapse-all control had to answer a question that isn't as obvious as it
sounds: what should it say, and do, when the sidebar is in some mixed state — half folded, half
open? The answer that actually matches how someone would use it: the button's only job is to finish
the job. If anything at all is still folded, it offers to open everything, remaining sections
included, not just the ones already open. Only once nothing is left closed does it turn into an
offer to close everything instead. Read the whole list, decide which of the two possible actions is
actually still useful given what's currently on screen, and stay that way — not two separate
buttons, not a fixed label, just a control that keeps asking the right question, including while a
live search is temporarily forcing some of that state open on its own.
