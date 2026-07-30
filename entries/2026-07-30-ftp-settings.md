# Phase 36 - FTP Settings, and a Sandbox Gap That Predated It

A follow-up to this morning's FTP accounts work: Service Status now reports vsftpd's live state
alongside every other daemon, and there's a new FTP Settings page for the server-wide knobs —
connection limits, idle session timeout, upload umask, passive port range, welcome banner. Same
edit-form-then-apply shape as Nginx Settings and PHP Settings.

vsftpd doesn't give you the two things that pattern normally leans on. There's no `nginx -t`
equivalent — no dry-run or syntax check — and no include mechanism, so a single small snippet can't
be regenerated in isolation the way an Nginx vhost's client-body-size directive can. The whole
config gets rebuilt instead: fixed architectural directives (SQL-backed auth, chroot, TLS
enforcement) and the settings this page exposes, together, every time. Validation became "did the
service actually come back up" — checked with `systemctl is-active` after a restart rather than
before one — with the same revert-the-backup-on-failure guarantee as every other config write in
the provisioning script.

Live-testing the actual save button (not a shell test standing in for it) surfaced something that
had nothing to do with FTP settings specifically: the request came back with `cp: cannot create
regular file '/etc/vsftpd.conf.bak': Read-only file system`. php-fpm's systemd unit sandboxes its
own mount namespace — `ProtectSystem=full` — and only unlocks specific `/etc` subtrees for the app
to write into. `/etc/vsftpd` was never added to that list when this morning's FTP accounts feature
shipped, so the account-creation flow had only ever been proven out from a direct root shell over
SSH, which doesn't run inside that sandbox and so never would have shown the gap. The real HTTP path
through the app had been silently broken since this morning. Fixed by adding `/etc/vsftpd` to the
unit's `ReadWritePaths`, and relocating the settings backup file itself from directly inside `/etc`
(a single-file grant like `/etc/vsftpd.conf` doesn't cover a *new* sibling file next to it) into
`/etc/vsftpd/` proper, which is now writable as a whole directory.

Re-verified for real after the fix: an admin login, a genuine form POST with changed values, a
banner change confirmed live over an actual FTP connection to the server, then reset back to
defaults the same way. Every value round-tripped through the browser, the database, the regenerated
config file, and the running daemon.
