# Phase 35 - FTP, Built the Same Way Mail Already Was

The one thing that had never been touched at all: FTP. Today closed that gap, and it turned out to
be less new ground than expected — the mail server had already solved the hard part.

**Every FTP account is a "virtual user," not a real Linux login**, authenticated against the
panel's own database using a restricted read-only database user, the same shape as how mailboxes
are authenticated. There's no separate user store to fall out of sync with what the app thinks
exists.

Every FTP session runs as the account's own system user, not a shared placeholder. A file uploaded
over FTP is therefore owned by, and counted against the disk quota of, the same user that runs the
account's website. Nothing FTP-specific had to be built to keep it consistent with the quota work.

Every account gets one FTP login automatically when it's created, with full access to its own home
directory. A new page lets the account owner create additional logins scoped to a subdirectory,
for handing out upload access without handing out everything, and confined there at the server
level rather than only by an app-level check.

One real thing caught during live testing on a development server: a configuration setting whose
name suggests it controls real system logins also governs virtual-user support, so copying a config
from memory instead of testing it caused every connection to be refused. Real system logins remain
blocked.

Verified end to end on a development server: a TLS-verified upload as the primary account, file
ownership matching what quota enforcement expects, a path-traversal attempt coming back empty, and
a scoped sub-account seeing only its own subdirectory.
