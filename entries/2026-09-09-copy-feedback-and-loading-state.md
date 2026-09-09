# Phase 210 - Nothing Happened, As Far As Anyone Could Tell

Two buttons, two different jobs, the same quiet complaint underneath both: click it, and nothing
visibly happens. Not because the click failed. Because whatever it actually did left no trace
anyone watching the screen could see.

## A button that already worked, silently

The Copy button next to a freshly generated password did exactly what it promised — the password
really was on the clipboard the instant it was clicked. Nothing on screen agreed. No color change,
no message, not even a flicker. Someone unsure whether a click landed does the natural thing: click
it again. And again, if the doubt doesn't clear. The fix isn't the clipboard call, which was always
fine — it's the half-second of proof that was missing, a small word appearing right where the eye
already is, gone only once the page moves on to something else.

Four separate pages had grown this exact same button, byte-for-byte identical, each written on its
own. Rather than teaching four copies the same trick, this became one small rule instead — any
button that says which element to copy gets the same behavior automatically, the same way a
confirmation prompt already works from one shared switch rather than a repeated block on every
form. The four pages shrank to pointing at that switch instead of carrying their own copy of the
logic.

## A button whose work takes longer than a click

The second case looked identical from the outside — click, then silence — but the actual cause was
almost the opposite. This button's job was slow, not silent: checking every linked server for
pending updates genuinely takes real time, sometimes the better part of a minute. The click did
register immediately. There was just nothing on screen distinguishing "still working" from "did
that do anything at all," so the wait read the same as the other bug even though nothing here was
actually broken.

The honest fix was a label, not a fake one: swap the button's own text to say what's actually
happening, with the same small spinner already used everywhere else in the panel something is
mid-flight, the instant a submit genuinely goes through rather than the instant it's merely
clicked. That distinction mattered more than it first looked — a button that also asks "are you
sure?" first fires that same click event twice, once for the question, once for real, and only the
second one should ever start the spinner. Catching that correctly meant reading a plain, boring
signal already sitting on every browser event — whether anything upstream had already stepped in
front of this particular attempt — rather than trying to track the confirmation dialog's own state
by hand and risk two pieces of code disagreeing about which attempt was the real one.

Small fixes, both of them. The kind that don't change what a feature does, only whether the person
using it can tell that it's doing it.
