# Phase 35 - FTP, Built the Same Way Mail Already Was

The one thing that had never been touched at all: FTP. No daemon in the stack, nothing on the
roadmap beyond a placeholder row. Today closed that gap, and it turned out to be less new ground
than expected — the mail server had already solved the hard part.

**Every FTP account is a "virtual user," not a real Linux login**, authenticated straight against
the panel's own database — vsftpd's PAM module queries a new `ftp_accounts` table directly through
a dedicated, read-only MySQL user, exactly the same shape as how Dovecot already authenticates
mailboxes. No separate flat-file virtual-user database to ever fall out of sync with what the app
thinks exists.

The detail that made this worth building properly rather than bolting on: every FTP session runs
as the account's own real system user, not some shared placeholder. That's a deliberate choice —
it means a file uploaded over FTP is owned by, and counted against the disk quota of, the exact
same Linux user PHP-FPM already runs that account's website as. Nothing FTP-specific had to be
built to keep it consistent with the quota work from a few days ago; it just already is, because
it's the same user.

Every account gets one FTP login automatically the moment it's created — full access to its own
home directory, matching cPanel's own default. A new page lets the account owner create additional
logins scoped to just a subdirectory, for handing out upload access without handing out
everything — jailed there by a real chroot, not just an app-level check, so there's no path out of
it regardless of what the underlying Linux permissions would otherwise allow.

Caught one real thing during live testing on the actual dev server, not just a local check: vsftpd
refused every single connection with "both local and anonymous access disabled," which reads like
it's about real system logins but isn't — its guest/virtual-user support turns out to internally
reuse the same "local login" code path, just redirected, so that setting has to be on even though
real Linux account logins stay fully blocked by the PAM configuration underneath it. An easy trap
to fall into copying a config from memory instead of testing it for real.

Verified end to end against the live server, not just in tests: a real TLS-verified upload as the
primary account, confirmed the uploaded file's ownership matches exactly what quota enforcement
expects, a path-traversal attempt trying to climb out of the account's own directory came back
empty, and a scoped sub-account was confirmed to see only its own subdirectory, nothing above it.
