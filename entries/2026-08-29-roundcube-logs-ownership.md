# Phase 144 - The Directory Roundcube Could Never Write To

First real use of yesterday's diagnostics page, and it paid off immediately. Loaded it against the
live Mail Only satellite and the directory listing gave the answer straight away:

```
=== /var/lib/roundcube/logs/ directory listing ===
total 8
drwxrwx---  2 www-data adm    4096 Jun 19  2025 .
drwxrwxr-x 16 root     syslog 4096 Aug 29 12:39 ..
```

`www-data:adm`. Debian's `roundcube-core` package default. Not `root:roundcube`, which is what
`setup-mail-server.sh` is supposed to set, and which the Roundcube PHP-FPM pool actually runs as.
That chown line has run on this server — it's right there in the script, right after the `roundcube`
user gets created — but it's wrapped in `2>/dev/null || true`, and something about this particular
install made it fail silently. Cause unconfirmed (a race with when the directory first gets created,
apt's own postinst re-asserting its default on some later step — doesn't matter which for the fix),
but the practical result was that the Roundcube pool user could never write to its own log directory,
on this server, this whole time.

That also explains something that had been nagging at the des_key fix from two phases ago: PHP-FPM's
own global log showed zero fatal errors across the entire window we'd been hitting `/roundcube/` and
getting 500s. Not one. A PHP fatal error normally shows up there. The likeliest reading is that
Roundcube's own error handler was catching whatever the real failure was — the des_key issue, or
something else entirely — and then failing *again* trying to write it to a log file it didn't have
permission to touch, with nothing left standing to report either failure anywhere. Two bugs stacked
exactly wrong: one that breaks a real thing, one that hides the evidence of the first.

Fixed both ends. `setup-mail-server.sh` no longer swallows a chown failure in silence — it still
doesn't abort the install over it (that's the right call, a logging directory isn't worth failing a
whole mail server setup for), but it says so now if it happens. And the same unconditional self-repair
step in `issue-panel-cert` that already re-checks the des_key on every call picked up a second job:
re-chown `/var/lib/roundcube/temp` and `/logs` back to `root:roundcube` every time, so a server that's
already carrying this exact problem repairs itself the next time anyone touches its hostname or
certificate, no SSH required.

Whether this alone clears the 500, or just finally lets the real error reach a log where the next
`roundcube-diagnostics` pull can actually see it — don't know yet. Applying this to the live stack and
checking is the next step, not this entry.

Tested (2506/2506).
