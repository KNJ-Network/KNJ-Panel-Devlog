# Phase 36 - FTP Settings, and a Sandbox Gap That Predated It

A follow-up to this morning's FTP accounts work: Service Status now reports the FTP service's live
state alongside every other daemon, and there's a new FTP Settings page for the server-wide
options — connection limits, idle timeout, upload umask, passive port range, welcome banner. Same
edit-form-then-apply shape as Nginx Settings and PHP Settings.

The FTP server has no dry-run syntax check, so the whole configuration is rebuilt on every save.
Validation became "did the service come back up" after applying, with automatic revert of the
previous configuration on failure, like every other configuration write in the panel.

Testing the actual save button on a development server surfaced something unrelated to FTP
settings themselves: the save failed with a read-only file system error because one configuration
location needed by the FTP accounts feature had not been permitted for the app. Earlier testing had
used a different path that didn't reveal the gap. It was fixed, and the lesson is to test through
the real application path, not only from a shell.

Re-verified after the fix: an admin login, a real form submission with changed values, a banner
change confirmed over a real FTP connection, then reset to defaults.
