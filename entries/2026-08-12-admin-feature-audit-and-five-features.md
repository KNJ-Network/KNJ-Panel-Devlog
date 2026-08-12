# Phase 72 - A Real Feature Audit, and the Feature That Said No to Itself

This phase went further than the roadmap document: a full re-audit of the admin feature catalog,
cross-referenced against the live codebase. Most of it held up. Two items didn't: List Parked
Domains/List Subdomains and Spamd Startup Configuration were filed as "Not planned, blocked on X",
and X had since shipped. Two more had never been tracked at all: Edit Database Configuration and a
proper Terminal.

The two stale notes were fixed on the spot. Then four real gaps were built, plus one more the
sweep surfaced as an open question: a Security Policies overview.

## The four straightforward ones

**List Parked Domains** is an admin-side rollup across every account's addon domains. The panel
had no separate "subdomain" concept at the time, since every extra domain was a first-class site
record, so "List Subdomains" didn't apply.

**Spamd Startup Configuration** covers children, timeouts, allowed addresses, local-only mode,
debug logging, and nice level for the spam daemon, separate from the scoring page. Nice level
turned out not to be a daemon flag at all, so it's applied as a process priority instead.

**Edit Database Configuration** is a deliberately small slice (connection limits, buffer pool,
packet size, table cache), not full exposure. Every save restarts the database, and the UI says so
before the button is pressed.

**Security Policies** is a roll-up of login lockout status, password policy, host access rules,
and 2FA adoption, plus one new capability: requiring 2FA rather than leaving it opt-in.

## The one that pushed back

Terminal was the fifth item. A real terminal is, by definition, a root shell, and the panel was
designed so that the web application itself does not run with elevated rights. Building a terminal
would have meant a broad new grant on the one feature built to run arbitrary commands.

We weighed three options and chose the safe one. Diagnostics shipped instead: a fixed set of
read-only checks (uptime, disk usage, memory, listening ports, service status and logs) that cover
the need to check the server without opening SSH. This was a deliberate architectural choice
rather than a corner cut by accident.

All five were tested (1116/1116 in the full suite), linted, and documented on the roadmap with
the reasoning behind each call.

## One more bug, found the same way

Live-verifying Spamd Startup Configuration through the real UI found a bare "Read-only file
system" error when saving, a sandbox allowance missing for the new configuration location, fixed
the same way as earlier instances. The Database Configuration save was also verified live (a real
restart, with the database back up cleanly), as was the Require-2FA toggle. Turning that on for a
test admin without 2FA correctly restricted it to its own profile page, which is the feature doing
what it says.
