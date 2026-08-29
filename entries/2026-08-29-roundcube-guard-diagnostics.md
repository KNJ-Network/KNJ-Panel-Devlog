# Phase 145 - Trusting the Guard Less

Applied yesterday's ownership fix to the live Mail Only satellite via "Set Hostname & Issue
Certificate" — twice, a minute apart, on the theory the first attempt might have raced the panel
update's own queue-worker restart. Neither one changed anything. The diagnostics page's directory
listing came back byte-for-byte identical both times: `www-data:adm`, same `Jun 19 2025` timestamp
on the directory itself. Not a partial repair, not a permissions wobble — nothing touched it at all.

Read back through `issue-panel-cert`'s own code to make sure the self-repair block was really where
the comments say it is: unconditional, outside the marker gate that skips re-running the full mail/DNS
setup, gated only on `PANEL_INSTALL_ROLE` being `main` or `mail_only` and
`/etc/roundcube/config.inc.php` existing. On paper it should fire on every single call. It clearly
isn't, on this server, or it's firing and failing at the `chown` itself for a reason `2>/dev/null ||
true` is swallowing just as effectively as the original bug did.

Both are guesses. Neither is worth shipping a fourth blind fix for. The diagnostics page already
proved once that building a small, permanent, read-only window beats iterating in the dark — so
`roundcube-diagnostics` gained a second section: the exact inputs to that guard, printed straight off
the box. The role read from `.env`, `config.inc.php`'s existence and ownership, `id roundcube` /
`getent group roundcube`, the `des_key` file's byte length, and a listing of `/var/lib/roundcube/temp`
alongside the `/logs` listing it already had. No behavior change anywhere — this is purely instrumentation,
the same category of change as the diagnostics page itself.

Next pull of that page tells us which half of the guard is lying, or whether the guard is fine and the
`chown` itself is failing on something more specific than a missing group. Either way, guessing again
before that pull would just be the same mistake with extra steps.

Tested (2506/2506).
