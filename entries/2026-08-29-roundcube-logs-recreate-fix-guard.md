# Phase 148 - The Fix That Broke Itself

Applied the recreate-on-failure repair from the last entry, expecting either a fixed directory or at
least a fresh mtime proving the attempt happened. Got neither. The diagnostics page came back with the
exact same listing as before — `www-data:adm`, `Jun 19 2025`, unchanged down to the timestamp. Not "the
recreate ran and produced the same result" — "the recreate never ran at all."

The answer was sitting at the top of the script the whole time: `set -euo pipefail`, line 166. Every
other self-repair line in this action — the `chown`, the `chmod`, the `des_key` regeneration — ends in
`2>/dev/null || true`, and that's not decoration. Under `set -e`, any command that exits non-zero kills
the whole script unless something explicitly catches it. The new recreate block — `rm -rf`, `mkdir -p`,
`chown`, `chmod` — had none of that. Four bare commands, and the first one operating on the one
directory this entire investigation has already shown behaves strangely under basically every other
tool that's touched it.

If `rm -rf` failed on `/var/lib/roundcube/logs` — and given `lsattr` couldn't even read its flags,
that's not a stretch — the script would have stopped dead on that line. Not just the recreate: the
entire rest of `issue-panel-cert`, including whatever ran after it in the same action, silently didn't
happen either. A fix for one bug quietly introduced a worse one: a self-repair that, when it hit the
one case it actually needed to handle, took the rest of its own action down with it.

The correction is unglamorous — `|| true` on all four lines, matching the convention that was already
sitting right next to the new block, ignored by mistake rather than by decision. Whether the recreate
itself actually succeeds against whatever's wrong with this directory is still an open question. What's
no longer open is whether a failure there can silently swallow everything downstream of it — it can't,
now.

Tested (2506/2506).
