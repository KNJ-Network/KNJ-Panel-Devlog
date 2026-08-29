# Phase 149 - Asking Instead of Guessing

Every version of this self-repair, going back to the very first one, ran its `chown` with
`2>/dev/null`. That wasn't sloppiness — it's this codebase's own established convention for
self-repair steps that are supposed to be cheap and silent, and every other self-repair line in this
exact action follows it too. But it meant that four straight releases chasing this one directory
(ownership fix, guard-inputs, `lsattr`/`findmnt`, recreate-on-failure, `set -e` guard) all had to
*infer* what was going wrong from side effects — an unmoved mtime, `lsattr`'s own failure — because
the one place that would have just said so outright was being thrown away every single time.

The guarded recreate from the last release should have been the end of it: `rm -rf`, `mkdir`, `chown`,
all protected against aborting the script, all run against a directory that should no longer exist by
the time `chown` gets to it. It changed nothing. Same owner, same timestamp. That rules out the
`set -e` theory as the *whole* explanation — the operations aren't failing because something upstream
died, they're failing on their own terms, repeatedly, and nothing has ever shown what those terms are.

So: stop inferring, start reading. Added a version of the same `chown` and a direct `touch`-based write
test to `roundcube-diagnostics`, this time with nothing swallowing the error text — whatever the real
message is, it lands on the page verbatim. Alongside it, a check for AppArmor: `aa-status` filtered to
anything mentioning roundcube, php-fpm, or nginx, plus a scan of the kernel log for recent
`apparmor="DENIED"` entries. `lsattr` returning "Operation not supported" instead of a permission
error was already the detail that ruled out `chattr` immutable a few phases back; a directory that
resists `chown` even freshly recreated, on a path that isn't a distinct mount, is exactly the shape a
path-scoped mandatory-access-control policy would produce — restricting the *path*, not whatever inode
happens to be living at it.

Whatever the next pull of this page says, it'll be the first time this investigation has read an
actual error message instead of reasoning around the absence of one.

Tested (2506/2506).
