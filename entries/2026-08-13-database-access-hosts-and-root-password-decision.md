# Phase 77 - A Feature That Wasn't There to Build

Next on the gap list: a database root password rotation feature. Checked how this server's own
MariaDB root actually authenticates before writing a line of code, and the honest answer changed
the plan.

## Nothing to rotate

Local root on this server has no password at all — it authenticates through the Debian/Ubuntu
default `unix_socket` plugin, which ties database access to already having a root shell on the box.
There's no credential sitting in a config file, no `/root/.my.cnf`, nothing a UI could safely
"rotate." That's not really a gap against the parity checklist, it's a stronger starting
position: a network-facing password can be phished, brute-forced, or leaked in a log; a socket-only
credential can't be presented over the wire at all. Adding a password anyway, just so there's
something to rotate, would mean introducing a new secret into a system this server's own
provisioning script already depends on staying passwordless — a real chance of locking the server
out of its own database for a feature that makes nothing more secure. Flagged this to the user
before deciding, and the call was to mark it not applicable rather than force a feature to exist
where the premise doesn't hold.

## The item that actually was missing

Right next to it on the list: "Manage Database Profiles / Access Hosts" — and this one was real.
Account owners have always been able to allow a specific remote IP to connect to their own database
users, self-service, from their own Databases page. What was missing was the admin side: a
server-wide view of every database user across every account and the hosts each is allowed to
connect from, with the ability to grant or revoke one directly, without going through the account
owner. Built it reusing the exact same two service methods the self-service page already calls —
`allowRemoteHost()` and `revokeRemoteHost()` — with zero new logic to keep in sync between the two.
Live-verified against a real database user and a real remote grant: the MySQL `'user'@'host'`
account genuinely appeared and genuinely disappeared, not just the panel's own row for it.

## Next

Continuing down the list — install a Perl module from the admin side, then a real Feature Manager
page.
