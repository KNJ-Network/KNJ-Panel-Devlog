# Phase 42 - The Release Pipeline, and the Three Bugs It Took to Prove It

Every update mechanism this panel has shown off up to now — the version ticker, the "update
available" banner — was checking against nothing real. There was no actual place a release could
come from, and no actual way to cut one. That gap sat quietly behind an otherwise-finished feature
for longer than it should have.

It's closed now: a dedicated build repo the panel's own install and update scripts pull from, a
sync job on the licence server that keeps it current within a minute of a real release being cut,
and a one-click "Update Now" button on the Panel Updates page with the same live terminal-style log
already used for OS package updates — or the exact same upgrade run over SSH by hand, for anyone
who'd rather do it that way.

None of that would have meant anything proven from a green test suite alone, so it got tested the
only way that actually counts: for real, on the real dev server, watched live rather than trusted.
That's what found three separate bugs, each one only visible once the whole thing actually ran
end to end.

The first: the upgrade script restarts the panel's own background worker as its last step. Run it
from the Panel Updates page and that worker is the very process running the upgrade — a plain
restart kills its own executor mid-run, before it can report back that anything succeeded.
`--no-block` looked like the fix and wasn't; systemd still tears the old worker down within a
second or two regardless of whether the command that triggered it waited around. What actually
works is scheduling the restart as a fully independent unit, decoupled from the worker it's
replacing, so tearing the old one down can never reach the thing reporting the result.

The second was subtler: the version number the panel displays comes from a config value that gets
compiled into a static cache — and that cache was being rebuilt *before* the new version file was
written, not after. The upgrade would genuinely, completely succeed, and the panel would keep
showing the old version anyway, forever, with nothing left to ever notice or fix it. Reordering two
lines was the entire fix, once the actual cause was clear.

The third wasn't a bug in the panel at all — it was a process gap. A release gets built from
whatever's committed at the moment the build is cut; an uncommitted local change is silently just
never included, no error, nothing to notice until someone wonders why a feature they just wrote
isn't in the build they just shipped. The fix was making that impossible to do by accident: cutting
a release now refuses outright unless the source is clean and fully pushed first.

Fourth time was clean, start to finish, version number updating correctly the moment the run
actually finished — not before.
