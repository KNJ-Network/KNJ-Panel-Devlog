# Phase 48 - A Firewall Bug That Only Failed Through the Real App

Two items shipped in the Security section this session: per-service IP allow/deny lists
("Access Control") and a central, admin-configurable password policy. Two more — scoped API
tokens and external login — are still in progress and picked back up next session.

Password Policy was the straightforward one: a single `PasswordPolicyService` reading from
Settings now builds the password validation rule everywhere a password gets set on this server,
replacing a literal `min:12` that had been copy-pasted into roughly ten different controllers.
Defaults deliberately match what was already enforced, so no existing install's behavior changes
until an admin actually turns on stronger requirements.

Access Control — restricting Panel Login, SSH, FTP, Mail, or DNS to a specific allow-list of IPs
at the firewall level — was a different story entirely. The feature worked perfectly every time it
was tested by hand over SSH, and failed silently almost every time it was triggered through the
real running app: the IPv6 side of the rule would land, the IPv4 side just wouldn't, with no
error anywhere. Chasing it meant ruling out one plausible-sounding theory after another —
a conflict with ufw's own baseline rules, a race between concurrent firewall writes, systemd's
sandboxing on the PHP-FPM service, even apply latency under that sandbox — each one patched,
each one still leaving the bug in place.

The actual cause turned out to be almost embarrassingly simple once found: the list of allowed
IPs gets written to a file from PHP using `implode("\n")`, which never adds a trailing newline
after the last address. A bash `while read` loop's exit status is non-zero for a final line with
no trailing newline — and a loop written as `while read -r VAR; do ... done < file` never runs its
body for that line at all, even though `read` did capture the value. Every manual test during
debugging used `echo` to write its test file, which always adds a trailing newline — which is
exactly why the bug could never be reproduced by hand, only through the app that had written the
file for real. Fixed with the standard shell idiom for exactly this case
(`read ... || [[ -n "$var" ]]`), verified with several full cycles through the real UI afterward,
both directions, before calling it done.

The defensive extras added while chasing the wrong theories — serializing firewall writes with a
lock file, verifying every rule actually landed before reporting success — stayed in. They weren't
the fix, but they're real insurance against a tool (ufw) that has genuinely demonstrated more than
one way to report success without doing what it said.
