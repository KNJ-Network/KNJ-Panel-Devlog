# Phase 250 - The Link That Was Fine

A safety check that rejects something harmless is a bug, even if it's the safe kind of bug — because
the cost is a restore that fails on the one day someone really needs it.

## A restore that refused its own customer

Restoring an account from a backup is a sensitive step, and the archive is checked before anything is unpacked. One of the checks was stricter than it needed to be.

Run against a realistic backup, the restore failed immediately. The flagged item was an entirely ordinary shortcut found in a typical web project, and it never left the account. The rule was refusing harmless items along with dangerous ones.

## Fixing it without opening a door

Loosening a security check deserves suspicion, so the fix started from the risks, not the convenience. The checker now works out where an item really leads and refuses only what leaves the account, and the surrounding rules were tightened to match.

The real checking function is now run against real archives in the test suite: ordinary items accepted, and anything reaching outside the account refused.
