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
Then it asked what a determined owner could do with the new freedom:

- A link allowed to point "up" could be paired with a second link that walks through the first, and a
  file written beneath that second link would land outside the account. So `..` is only allowed at the
  start of a target, never after a folder name, and nothing in an archive may be written beneath a link
  from that same archive.
- While testing the hard-link case, a pre-existing hole turned up: the archiving tool quietly removes a
  leading `../` from a hard link's target when it lists the archive — and when it unpacks it. A hard
  link to another account's file therefore looked like an innocent plain path to the old check. The rule
  no longer depends on spotting `..`: a hard link may only point at something inside its own top-level
  folder.

The real checking function is now pulled straight out of the privileged script and run against real
archives in the test suite: ordinary links accepted, links out of the account refused, the two-link chain
refused, writing beneath a link refused, and hard links to other accounts refused.
