# Phase 189 - The Page That Grew Instead of Scrolling

Two people looked at the same inbox and saw two different problems. One saw a sidebar that kept
sliding out of reach as messages piled up. The other saw an address book that genuinely had nothing
in it, on a server where the same address book, reached a different way, was full. Neither problem
was where it looked like it was.

## Growing instead of scrolling

A page that's too tall to fit the screen has two honest choices: let the whole page get taller than
the window and scroll as a unit, or fix the window's own height and let one part of it scroll on its
own while the rest — a sidebar, a header — stays put. This page was built to do the second thing and
was quietly doing the first instead, because of one word. It was told its own height was a *minimum*,
not a fixed amount — which sounds like a small difference, but a minimum is free to grow past the
screen the instant its content needs the room, taking the whole layout down with it. Everything below
that top-level choice inherited the same permission to keep growing rather than stop and scroll, all
the way down to the one part — the message list — that was supposed to be the only thing moving.

Fixed by pinning the outer shape to the screen's real height instead of a floor it could grow past,
then explicitly telling each nested section along the way that it's allowed to shrink to make room
for that promise, since that's not something a browser assumes on its own for a box sized this way.
The sidebar's own contents — folders, contacts, settings, the logout button — stopped drifting off
the bottom of the screen the moment the page itself stopped being taller than the screen at all.

## A folder that only reappeared on the way to writing an email

The Contacts page had a stranger version of the same shape of bug. Every other page in this app kept
the sidebar's folder list visible the whole time. Contacts didn't — visit it, and the folder list
vanished outright, taking the only obvious way back to the inbox with it. The one workaround anyone
had found by accident was to open Compose first, which brought the folder list back, and from there
you could get back to Inbox normally. Contacts itself just never asked for the list at all — it
handed the layout an empty one on purpose, as a leftover from before this page needed to share that
part of the screen with anything real. Every other page already knew how to ask correctly; this one
simply never got taught the same thing when it was built.

## The address book that was full in one place and empty in another

The deeper issue took longer to see, because everything about it looked like success. A background
sync between the two ways of reading mail on this server is supposed to keep one shared address book
current in both places, and by every visible signal it was doing exactly that — no error banner, no
warning anywhere obvious, a request that came back looking entirely normal. And yet, on a server where
mail runs on its own separate machine rather than the main one, that address book stayed empty on one
side no matter how many times the sync ran.

The request meant to carry that sync across to the separate mail machine was being turned away before
it ever reached the part of the system meant to run it — a newer kind of request that machine hadn't
yet been told to expect, so it landed exactly the way an unrecognized login attempt from a browser
would: politely redirected toward a sign-in page instead of refused outright. That redirect still
counted as a normal, complete response from the sending side's point of view — nothing about it looked
like a failure worth raising an alarm over, so none was raised. It only stopped looking like success
once someone went and read exactly what came back, line by line, instead of trusting that a clean
response meant a clean result.

Two things needed fixing, not one. The obvious one: teach that separate machine to recognize this
particular kind of request as legitimate, the same way it already recognizes every other kind it's
meant to handle. The quieter one, worth just as much: make the sending side spell out clearly that
it's expecting a real, structured answer back — not a page meant for a browser — so that if something
like this ever slips through again, it fails in a way that's impossible to mistake for success,
instead of getting politely waved through toward a door that was never the right one.

## Checking a real address book, not a rehearsal of one

Once both fixes were in place, the actual account this was first noticed on was checked directly, not
just trusted to be fixed because the code now looked right — and the numbers matched afterward,
address book to address book, both sides finally reading the same list.

Tested (2777/2777).
