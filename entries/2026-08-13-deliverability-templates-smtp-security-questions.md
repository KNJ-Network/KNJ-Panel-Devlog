# Phase 75 - Working Down the Gap List

Back to the open items from the earlier click-through comparison, taken from the top of the list:
a client-side deliverability checker, a stale tracking note, a real web page template editor, an
outbound port-25 lockdown, and an unfamiliar-IP login challenge.

## Email deliverability, for real domain owners this time

The admin side already had a per-account deliverability check (SPF/DKIM/DMARC/PTR, built earlier
this session). The client side didn't — a domain owner had no way to see their own mail
authentication status without asking support. This reuses the exact same DNS-lookup service
underneath (a real check, not a second implementation of the same logic) and adds a Repair button
per domain, wired to the same DNS zone service that already knows how to write correct
authentication records. No new privileged server action was needed at all — DNS lookups are just
reads, and repairing a zone was already a solved problem elsewhere in this codebase.

## One line that was quietly wrong

A private tracking note claimed the mail configuration manager was still missing a couple of
tabs. Both had actually shipped since that note was written — a stale checkbox, not a real gap.
Fixed the record, moved on.

## A real web page template editor, and a sandboxing bug worth writing down

Suspended and default placeholder pages were previously fixed HTML baked into the provisioning
script. Now an admin can edit both from the panel itself, with a live preview, and a "restore
built-in default" button that's actually reversible.

The interesting part was getting the panel's own PHP process permission to write the new config
directory at all. This server's PHP-FPM pool runs under a locked-down systemd sandbox — most of
`/etc` is read-only except an explicit allow-list. Adding the new directory to that allow-list
looked like the whole fix, and even survived a restart looking fixed... and then failed again,
identically, the next time it was actually used. The real cause: that allow-list can only grant
write access to a directory whose *parent* already exists on disk at the moment the sandboxed
process starts. Every existing entry on that list has a parent that some system package already
created. This was the first genuinely new top-level directory anyone had asked the sandbox to
write to — no restart, however many times, was ever going to fix it, because the parent still
didn't exist before the process launched. The actual fix was one line in the server's own
provisioning script, creating that directory ahead of time, matching the "no operator has to SSH
in" rule this panel is built around.

## Stopping the quiet way spam actually leaves a compromised account

A compromised account's PHP or cron script doesn't need this server's own mail queue to send spam
— it can just open its own connection straight out on port 25 and bypass the queue, the rate
limits, and DKIM signing entirely. A new toggle blocks exactly that: every process on the server
except the mail server's own delivery agent is denied outbound port 25. Genuine mail sent through
this server's own mailboxes is completely unaffected — verified live, end to end, including a
real delivery attempt from the mail server itself succeeding while an unrelated process got
`Network is unreachable` for the identical attempt. The one file this touches — the firewall's own
rule set — is checked for valid syntax before it's ever reloaded, with an automatic revert to a
real backup if anything about the change looks wrong.

## An extra check when a login doesn't look like you

The last item: an unfamiliar-IP login now gets interrupted with a security question, on top of
whatever else is already required. Each admin or reseller sets their own question and answer;
turning the whole thing on is a separate, org-wide switch, off by default. A user with nothing
configured is never challenged, even while it's switched on.

The one thing worth being careful about: a login that already uses two-factor authentication
shouldn't lose that protection just because this new check runs first. A correct answer here
doesn't finish logging anyone in by itself — it hands off into whichever check the user already
had configured next, the same way this panel's existing authenticator-app, recovery-code, and
emailed-code paths already hand off to each other. Only once every configured check has passed
does the session actually start.

## Next

Continuing down the same list — an SSL certificate storage manager, a database root password
reset, and per-database access profiles are next.
