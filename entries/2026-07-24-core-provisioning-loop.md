# Phase 04 - The Core Loop Actually Works

The milestone that actually proves this is a real hosting panel and not just an admin UI:
creating an account now provisions a real, isolated hosting environment. Not a mock, not a
placeholder — a genuine Linux user, an Nginx site, a dedicated PHP-FPM pool, and its own MySQL
database, torn down cleanly when the account is removed.

## What actually happens

Create an account through the controller UI, and in the background:

- A real Linux user gets created — no login shell, isolated home directory
- An Nginx server block goes live for the domain
- A dedicated PHP-FPM pool, running as that account's own user (not shared with anything else)
- A MySQL database and user, scoped to just that database

Verified by actually visiting the provisioned site and getting real PHP output back, not by
trusting a green checkmark.

## Two things worth being honest about

**A real bug, caught before it mattered**: provisioning initially ran synchronously inside the
web request. The provisioning script needs to reload PHP-FPM after writing a new pool config —
except that reloads *all* pools, including the one serving the request that triggered it. First
few attempts came back as 502s. Fixed properly: provisioning now runs as a background queued
job, with its own dedicated worker process. Better architecture regardless of the bug it also
happened to sidestep — a slow, system-mutating operation has no business blocking a web request.

**A feature deliberately not built**: File Manager was part of this milestone's original scope.
Didn't rush it in. Direct file access needs careful protection against path traversal, and that
deserves its own focused pass rather than being the last thing bolted onto an already-large
milestone at the end of a long session.

## Security shape, since it matters here

Every privileged operation (creating system users, writing root-owned config, touching the
database) goes through exactly one audited script, granted exactly one narrow permission — not
a spray of individually-permitted raw commands. All input validation happens inside that script
itself. System usernames are always generated internally, never derived from anything a user
typed in — which is what makes it safe to use them in privileged commands at all.

## Next

File Manager, then SSL automation (M3) so provisioned sites actually get HTTPS automatically.
