# Phase 308 - Is FTP Safe

FTP Settings was a short form in a narrow box, with nothing to say whether FTP was actually safe or even in use. Looking at the
real server turned up the answer to that: it was still accepting the long-obsolete TLS 1.0 encryption, there was no way to
tell the server its public address when it sat behind a router, nothing limited how fast one customer could transfer, and
the log of who had been trying to log in was something nobody could see. That log was full of bots, all day, every day.

## What changed

The page now opens on whether FTP is working and safe, and lists anything that needs attention with a button or a link to fix
it: old encryption, unencrypted logins, a firewall that blocks the file-transfer ports, a certificate about to run out, a
customer's password being guessed. There is a week of activity at a glance and a list of who has been trying to get in.

Many more settings are in reach: the oldest encryption allowed, the public address for servers behind a router, timeouts, a
speed limit for each customer, and how strict to be about wrong passwords. Every server moves to modern encryption by default
when it upgrades, without disconnecting anyone unless something actually changed.
