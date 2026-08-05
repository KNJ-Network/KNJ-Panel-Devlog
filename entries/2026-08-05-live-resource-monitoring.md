# Phase 55 - Live Resource Monitoring, and Backup & Server Status Closed Out

One item left in this section: a real process list and disk usage view, with the ability to stop a
runaway process. The closest existing thing in this codebase was the MySQL-specific process list
and Kill button built for Database Maintenance Tools a couple of sections back — a good pattern to
mirror, but this needed to work at the OS level instead, across every process on the box, not just
MySQL's own connection list.

Listing turned out to need no privilege at all. Any user on a standard Ubuntu box can already see
every process system-wide via `ps aux` — that's the whole point of the command — and `df` reports
real filesystem usage without root either. Confirmed this directly against the live server before
building anything, rather than assuming: `sudo -u knjpanel ps aux` from the app's own unprivileged
user showed every other user's processes fine, PID 1 included. So the process list and disk usage
table both just shell out directly, no `sudo`, no new provisioning-script read action.

Killing is the one part that genuinely needs root, and it's also the one part worth being careful
with — a wrong click here could take the whole server down, not just one hosting account. New action
`os-process-kill <pid>` refuses PID 1 outright, then reads the target process's real name fresh from
`/proc/<pid>/comm` at kill time — never trusted from whatever the request claims — and refuses a
fixed list of anything the panel or every hosted site depends on: nginx, PHP-FPM, MariaDB, SSH,
BIND. Same shape as the existing Service Manager's own "can't stop what we depend on" rule, just
applied to process names instead of systemd units, and the message on refusal points the admin at
Service Manager's restart button instead.

Live verification was the actual test, same standard as always: started a genuinely disposable
`sleep 300` process on the real server, killed it through the panel, confirmed it was actually gone.
Then, deliberately, tried to kill nginx's real PID through the same button — refused, with the exact
message, and the site kept serving requests the whole time. That's the difference that matters here:
proving the guardrail holds against the one mistake this feature could actually make catastrophic,
not just that a happy-path kill works.
