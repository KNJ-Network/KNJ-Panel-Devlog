# Phase 250 - The Link That Was Fine

A safety check that rejects something harmless is a bug, even if it's the safe kind of bug — because
the cost is a restore that fails on the one day someone really needs it.

## A restore that refused its own customer

Restoring an account from a backup unpacks the whole archive as the system's most privileged user. An
account owner controls everything in their own home, so their backup can contain anything — which is
why that step runs the archive past a strict checker first. One of its rules: refuse any symbolic link
that points "up" a folder, because a link that climbs out of the account could let a later file be
written somewhere it shouldn't.

Run against a real backup of a real account, the restore failed immediately. The "dangerous" link was
the most ordinary thing in a Node project: a tool shortcut inside `node_modules/.bin` pointing at a
sibling folder one level up. It never left the account. The rule was refusing every link with `..` in
it, whether or not it actually went anywhere it shouldn't.

## Fixing it without opening a door

Loosening a security check is the kind of change that deserves suspicion, so the fix started from the
attacks, not the convenience. The checker now works out where a link really lands, starting from the
folder the link lives in, and refuses only a link that climbs above the account's own top-level folder.
Then it asked what a determined owner could do with the new freedom, and tightened the rules around it:
links may only point "up" at the start of a path, nothing in an archive may be written beneath a link
from that same archive, and the rules for links between files no longer lean on how the archiving tool
happens to print them. Testing this turned up a related gap in how one kind of link was checked; it is
closed in the same change.

The real checking function is now pulled straight out of the privileged script and run against real
archives in the test suite: ordinary links accepted, and every way of reaching outside the account
refused.
