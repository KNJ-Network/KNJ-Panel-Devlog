# Phase 205 - The Other Update Button Learns the Same Trick

Panel Updates could already push a panel software update to every linked DNS-only and Mail-only server
from one screen. System Updates, the neighboring page that manages OS package patches, never got the
same treatment and only handled the server you were logged into.

We decided it should work the same way: a button to check every linked server at once, a table showing
pending updates and whether a reboot is needed, a button per row to update that server, and, unlike
Panel Updates, a button to reboot it too.

## Reusing the wiring, not reinventing it

The two features share almost everything that matters. Linked servers already authenticate in a way
that doesn't depend on a login session, and Panel Updates' cross-server flow is built on that: send a
command, then poll for status. There was no reason to invent a second way. The new feature reads almost
identically to the old one with the nouns swapped.

The one piece needing new thinking was learning how many updates a remote server has pending, and
whether it needs a reboot, cheaply enough to check a dozen in one click. That needed a small addition to
the same channel, and the answer is cached for half an hour since it changes slowly.

## The button that didn't have a twin yet

Rebooting had no precedent in Panel Updates, which only restarts its own process. An OS update can leave
a server needing a full restart, and a button for that already existed on each server's own setup page.
The new button does the same thing from wherever the update was triggered, instead of requiring a second
login. Since a restart mid-update is a bad idea, the request is refused while that server shows an update
in progress, not just greyed out on screen.

## What's the same underneath

The question "which servers is this panel managing, and are they reachable" was being answered in four
separate places in the code, each written slightly differently. It is now one thing.

Full suite green, pint clean, and then live verification against a real second server caught something
the tests couldn't: the new endpoints were missing from an exemption list that other endpoints on this
channel already carry. The automated suite doesn't exercise that check at all, so the gap could only show
up against the real thing. It was fixed, a second small release was cut, and a real check, a real batch of
package updates and a real reboot all worked.

This is exactly what the disposable second server is for.
