# Phase 147 - Not Explaining It, Just Ending It

`lsattr -d /var/lib/roundcube/logs` on the live satellite came back with an error that changed the
whole shape of this investigation: `Operation not supported`. Not permission denied. Not operation
not permitted. Not supported — the kind of error a filesystem gives when the thing you're asking for
doesn't apply to the object at all, not when you're not allowed to have it.

That killed the leading theory. `chattr +i` — the immutable attribute, the thing that would have made
a tidy story out of "some hardening pass locked this directory down and it's now biting us" — fails
with `EPERM`, "Operation not permitted." A completely different error. Whatever's actually wrong with
this directory isn't a permission a service account lacks. It's something about what kind of thing the
directory *is*, at a level `ls -la`, `chown`, `chmod`, and every other tool this investigation has
already reached for simply can't see.

`findmnt --target` on the same path came back attributing it to the plain root ext4 filesystem — no
separate mount, nothing exotic registered anywhere this shell's mount namespace can see. So it isn't a
deliberately-separate filesystem either, at least not one visible from here. Whatever it is sits
somewhere between those two negative results: not an attribute, not a mount, and still, for four
versions running, a directory `chown` walks right past.

At that point the honest move was to stop trying to explain it. Four releases in a row (`v0.16.95`
through `v0.16.98`) had each added another window onto this directory — logs, temp, guard inputs,
extended attributes, mount info — and every one of them came back clean or came back with an answer
that ruled out the last theory without supplying a new one. That's a real pattern worth recognizing
for what it is: not "almost there," but "this class of diagnostic has nothing further to say."

The fix that doesn't require understanding the cause: stop repairing the directory in place and just
replace it. `issue-panel-cert`'s self-repair now checks ownership after the `chown` attempt, and if
it's still not `root:roundcube`, removes the whole directory and recreates it from nothing — an
ordinary `mkdir`, an ordinary `chown`, an ordinary `chmod`. Whatever made the old inode refuse
`chown`, a brand new inode at the same path won't have inherited it. And there was never anything
inside worth keeping — the entire bug, from the first version of this chase, was that nothing has ever
been able to write there.

Applying this to the live stack and checking is, again, the actual next step — not this entry.

Tested (2506/2506).
