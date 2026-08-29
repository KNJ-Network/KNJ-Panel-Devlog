# Phase 146 - One of Two, and Only One of Two

The guard-inputs pull from Phase 145 came back clean across the board: role read as `mail_only`,
`config.inc.php` present with the right owner and mode, the `roundcube` user and group both real,
`des_key` a correct 24 bytes. Every input the self-repair depends on was exactly what it should be.
And yet the diagnostics page's own listing of `/var/lib/roundcube/temp` showed something that hadn't
been true an hour ago: `root:roundcube`. The chown had run. It had worked.

Just not on `/logs`. Same directory listing, same command output pulled in the same page load, and
`/var/lib/roundcube/logs` was still `www-data:adm`, same `Jun 19 2025` timestamp it's had since the
start of this whole investigation. One chown call — `chown -R root:roundcube /var/lib/roundcube/temp
/var/lib/roundcube/logs`, both paths in the same invocation, same process, same moment — quietly split
down the middle. One side worked. The other didn't move at all.

That's not "the guard didn't fire." The guard fired; the evidence is sitting right there in `/temp`'s
new ownership. This is `chown` itself refusing one specific directory while succeeding on its sibling
one line of `ls -la` away. GNU `chown` handles each path argument independently — a failure on one
doesn't stop it from trying the rest — so a single invocation silently doing one and skipping the
other isn't unusual behavior, it's just unusual for it to actually happen on a directory that, by
every listing gathered so far, looks completely ordinary.

The leading explanation for a directory that `chown` won't touch even as root, with nothing about it
looking wrong in any `ls -la`: the immutable or append-only extended attribute — `chattr +i` or `+a`.
Neither shows up in a permissions listing. Both make `chown` fail with `EPERM` regardless of who's
asking. It's the kind of thing security-hardening tooling reaches for specifically to stop a log
directory from being tampered with — which would be a reasonable thing for *some* directory on this
box to have, given how many audit rounds this project has been through. Grepped the whole `deploy/`
tree for `chattr` first, on the chance this codebase set it itself and forgot. Nothing. If it's there,
it got there some other way.

Added `lsattr -d` on `/logs`, `/temp`, and their shared parent to `roundcube-diagnostics`, plus
`findmnt --target` on `/logs` in case the real answer turns out to be a mount boundary instead of an
attribute. Still read-only, still no behavior change — the pattern holds: see the actual state before
writing a fix for state that was only ever assumed.

Tested (2506/2506).
