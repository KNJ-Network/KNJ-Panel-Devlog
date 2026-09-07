# Phase 205 - The Other Update Button Learns the Same Trick

Panel Updates has been able to push a panel software update out to every linked DNS-only and
Mail-only server from one screen for a while now — a button, a table of servers, a status pill per
row. System Updates, the page next to it that manages OS package patches instead of the panel's own
code, never got the same treatment. It sat there checking and applying updates for exactly one
server: whichever one you happened to be logged into.

Asked directly for the same thing, in the same shape: a button to check every linked server at
once, a table listing them with how many updates each one has pending, whether any of them need a
reboot to actually finish applying what's already installed, and a button per row to update that
one server — plus something Panel Updates never needed, a button to reboot it too.

## Reusing the wiring, not reinventing it

The two features already share almost everything that matters. Every linked server this panel
talks to authenticates the same way — a two-key handshake completed once at linking time, after
which every command sent to that server rides inside its own private, unguessable URL rather than a
login session. Panel Updates' whole cross-server flow is built on that channel: dispatch a command,
then poll a status endpoint every couple of seconds until it reports done. There was no reason to
invent a second way of doing that. The new feature reads almost identically to the old one with the
nouns swapped — a tracking row gets created for whichever server was told to update, a job asks that
server to start, a second job checks back on it repeatedly, and the page polls its own status
endpoint the same way the existing one always has.

The one place that genuinely needed new thinking: knowing how many updates are pending, and whether
a reboot is needed, for a server nobody is currently checking. The panel already knew how to answer
that question about itself. Answering it about a server three hops away over an authenticated
channel, cheaply enough to check a dozen of them in one click without anyone waiting around, needed
its own small addition to that same channel — and since the answer barely changes minute to minute,
the result gets kept for half an hour before asking again, same as it already does locally.

## The button that didn't have a twin yet

Rebooting a different button entirely was the part with no existing precedent to lean on. Panel
Updates never needed to reboot anything — a panel software update just replaces some files and
restarts its own process. An OS update can leave a server needing a full restart to actually finish
taking effect, and there's already a button for that, sitting on that server's own Server Setup
page, reachable only by logging into that box directly. The new button does exactly the same
thing — the identical one-minute-delayed restart — just reachable from wherever the update itself
was triggered from, instead of requiring a second login to a different admin panel first.

Since a restart mid-update is a genuinely bad idea — half-applied packages, a database left locked
by whatever was mid-write — the button refuses to fire at all while that same server shows an
update still in progress. Not just greyed out on the screen; asked to run anyway, the request itself
gets turned away before it ever reaches the server.

## What's the same underneath

One number keeps a growing table honest: how many servers is this panel actually managing right
now, and are they really reachable. That exact question — every server this box knows about, minus
the ones added but never actually finished linking — was already being asked and answered in four
separate places in the code, each written slightly differently. Worth becoming one thing before it
became a fifth.

Full suite green, pint clean — and then live verification, against a real second server for the
first time, caught something the tests structurally couldn't. Every one of the four new endpoints
came back with a plain "session expired" page instead of the answer it was supposed to give. Every
other endpoint like it in this codebase carries an explicit exemption — the whole point of this
channel is that it authenticates itself, in the URL, with no session behind it at all, so checking
for a session token that will never exist just fails the request outright. These four were never
added to that list. The reason the automated suite sailed straight through anyway: that same check
is switched off for every test in this project, on principle, since none of them carry a browser
session either — so this exact gap had no way of showing up until something used the real thing.
One line added per endpoint, a second small release cut, and the second attempt went straight
through: a real check against a real linked server, a real batch of pending packages applied and
mirrored back correctly, and a real reboot — down, then back up, every service healthy on the other
side.

This is exactly the case the disposable second server was built for. A five-minute detour, caught
and fixed before it ever had the chance to reach anything that mattered.
