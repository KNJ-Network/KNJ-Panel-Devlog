# Phase 42 - The Release Pipeline, and the Three Bugs It Took to Prove It

The update mechanisms this panel had shown — the version ticker, the "update available" banner —
weren't yet checking against a real release source. There was no place a release could come from
and no way to cut one.

That's closed now: a dedicated build repository the panel's install and update scripts pull from,
automatic syncing of new releases, and a one-click "Update Now" button on the Panel Updates page
with the same live terminal-style log used for OS package updates — or the same upgrade run over
SSH by hand, for anyone who'd rather.

A green test suite alone wouldn't prove any of this, so it was tested for real on a development
server and watched live. That found three bugs:

The first: the upgrade restarts the panel's own background worker as its last step, and when run
from the Panel Updates page that worker is the process running the upgrade, so it was interrupted
before it could report success. The restart is now scheduled independently of the process it
replaces.

The second: the displayed version came from a cached value that was rebuilt before the new version
was written, so an upgrade could succeed while the old version kept showing. Reordering two steps
fixed it.

The third was a process gap: a release is built from what's committed, so an uncommitted local
change would silently be left out. Cutting a release now refuses unless the source is clean and
pushed.

The fourth run was clean, with the version number updating correctly once the run finished.
