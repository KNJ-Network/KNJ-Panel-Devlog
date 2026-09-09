# Phase 207 - The Server That Wasn't In Its Own Table

Two update-related buttons for this box lived in two different places. One card, further down the
page, listed every *other* server and let each one be checked, updated, rebooted, all from the same
screen. A second, separate card handled updates for the one server actually being looked at right
now — this box itself — with its own buttons, its own layout, no relation to the first card at all.
The obvious question, once actually asked out loud: why isn't this server just a row in its own
table?

## Same row, different plumbing underneath

Folding the local server into the table didn't mean building a new kind of row — every column
already existed, they just needed a second source to draw from depending on which row they were
filling in. Pending count, whether a reboot is waiting, current status: each one already had a
"read this from a server three hops away" version and now needed a "read this from right here"
version sitting right next to it. The buttons were the same story from the other direction — Update
and Reboot already existed for this box, wired to routes that had been there for months; the table
just needed to point at the right one depending on whose row it was drawing.

The part that mattered most: none of the code watching for a run to finish and updating a row's
status pill needed to learn a second way of doing that. It already worked off a row's own tagged
identifier, matching whichever server the last known run belonged to — this box's own identifier
was just as valid an answer as any satellite's. Give the local row the same tag every other row
already carried, and the exact same watching code updates it too, without ever being told there's
anything different about this particular row.

## A card that used to always talk gets asked instead

The old package table sat there fully populated on every single page load, whether or not anyone
actually wanted to look at it right then. With every server now sharing one table, always fetching
every server's own list on every load stopped making sense — nobody's looking at more than one at a
time anyway. So the card that used to always talk now waits to be asked: empty by default, a single
line asking which server to show, and a click on that server's own new button fills it in on the
spot. Click a different server, it swaps. Nothing about it needed a page reload — the button already
knew exactly where to ask, the card already knew how to receive an answer, only the wiring between
the two needed writing.

For a server whose current list already lives in this box's own memory, filling that card in is
immediate. For a server on the other end of the wire, showing that same list without a fresh
round-trip needed one thing added to what already gets fetched during a routine check-everyone
sweep: the actual package list riding alongside the count that was already being sent back. The
count told you *how many* — now the full list comes along for free, sitting there cached and ready
the moment someone actually wants to see it.

## Two things caught before they shipped, one real and one not

Clicking through a server that hadn't been checked yet turned up a genuine mistake: the card's own
heading never updated to say which server the message was actually about, so an error for one
server showed up sitting under a stale title still naming whichever server had been looked at
successfully just before it. Fixed by having the error path update exactly what the success path
already updates, instead of only updating half of it.

The second thing looked like a bug and wasn't. A server elsewhere hadn't been brought up to this
same release yet, so the extra detail this update taught the checking mechanism to send along simply
wasn't there yet on that server's end — a real pending count next to an empty list, until that
server caught up. Worth noticing the difference: one was a genuine mistake sitting in the new code
itself, the other was two systems mid-handshake, on their way to agreeing, not disagreeing forever.
