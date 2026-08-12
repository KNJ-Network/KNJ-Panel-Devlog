# Phase 72 - A Real Click-Through of WHM, and the Feature That Said No to Itself

Asked to go further than the roadmap doc this time: an actual expand-all sweep of every category
and tool in trywhm.net's real sidebar — 132 named functions across 29 categories — cross-referenced
against the live codebase rather than trusted from memory. Most of it held up. Two items didn't:
List Parked Domains/List Subdomains and Spamd Startup Configuration were both filed away months ago
as "Not planned, blocked on X" — and in both cases, X had already shipped, weeks apart, without
anyone circling back to un-block the note it had blocked. Two more had never been tracked at all:
Edit Database Configuration, and real WHM's own Terminal.

Fixed the two stale ones on the spot. Then built all four real gaps, plus one more the sweep
surfaced as a genuine open question rather than a checkbox: Configure Security Policies.

## The four straightforward ones

**List Parked Domains** — an admin-side rollup across every account's addon domains, something
account owners could already see for themselves but had no server-wide equivalent. Turned out this
codebase never grew a separate "subdomain" concept distinct from an addon domain in the first
place — every extra domain is already a first-class `Site` row — so "List Subdomains" specifically
doesn't apply; there's no second list hiding behind it.

**Spamd Startup Configuration** — max/min children, timeouts, allowed IPs, local-only mode, debug
logging, and nice level for the `spamd` process itself, separate from Spam Filtering's own
scoring/tagging page. Nice level turned out not to be a `spamd` flag at all (checked `spamd --help`
directly rather than assume) — real WHM applies it as a process priority, so this does too, via a
systemd `Nice=` drop-in rather than anything spamd parses itself.

**Edit Database Configuration** — a deliberately small slice of `my.cnf` (max connections, InnoDB
buffer pool, max packet size, table cache), not full exposure — same restraint an existing
slow-query-log toggle already applied, just extended rather than abandoned. Every save restarts
MariaDB to take effect; the UI says so before the button is pressed, not after.

**Security Policies** — real WHM's version is a toggle panel for its own internal plugin framework;
almost everything it'd cover already had its own dedicated page here. So this became a genuine
roll-up (login lockout status, password policy, host access rules, 2FA adoption across every admin
and reseller) plus one real new capability none of those pages offered on their own: requiring 2FA
rather than leaving it opt-in. Enforced in the same middleware that already re-checks reseller
suspension on every request — the natural place, since it was already running there.

## The one that pushed back

Terminal was the fifth item, and the one that actually stopped and asked a question back.

Real WHM's Terminal is a literal root shell. That's a non-issue for WHM itself — `cpsrvd` already
runs as root; there's no boundary being crossed by adding a shell to a process that already has
full access. This codebase made a different, deliberate choice from its very first commits: the
panel's own app process runs unprivileged, and every single server-touching action goes through one
allowlisted script sudoers permits with specific, validated arguments — the entire reason a bug in
the Laravel app can't become a compromised server on its own. Building a real Terminal to match
would mean a new, broad sudoers grant undoing exactly that property, on the one feature that exists
specifically to run arbitrary commands.

Flagged it before building anything, with the actual tradeoff spelled out — three real options, not
a leading question. The answer: build the safe version. So Diagnostics shipped instead — a fixed
set of read-only commands (uptime, disk usage, memory, listening ports, service status, service
logs) run through the same allowlisted script as everything else, covering the real "check the
server without opening SSH" need without opening arbitrary execution to do it.

Worth writing down plainly, since it came up directly afterward: this isn't a corner this build cut
by accident. Real cPanel/WHM's "the whole app is root" model is a widely-known, often-criticized
weakness of the platform — a disclosed WHM vulnerability is instant, unconditional root, no second
step needed. This panel's split between an unprivileged app and one narrow, audited escalation path
is strictly more defense-in-depth than the product it's modeled on, not a gap in matching it. The
one feature where "as close to WHM as possible" and "as secure as possible" actually pulled in
different directions was resolved in favor of the property that's been true of every other feature
in this codebase since day one.

All five: tested (1116/1116, full suite), linted, roadmap updated with the actual reasoning behind
each call rather than just a checked box, `VERSION` bumped to 0.15.47-dev.

## One more bug, found the same way every one before it was

Live-verifying Spamd Startup Configuration through the real UI (not just the test suite) turned up
one genuine bug: saving a nice level failed with a bare "cp: cannot create regular file
'.../knjpanel-nice.conf.bak': Read-only file system". The exact same `ReadWritePaths`/
`ProtectSystem=full` class of gap this codebase has hit for every single new config-writing feature
since the original Domain Forwarding incident — php-fpm's own sandboxing didn't yet know about the
new `/etc/systemd/system/spamd.service.d` path. Fixed the same way every prior instance was: added
the path to the override, following the exact pattern already documented in that file's own header
comment. Also live-verified the Database Configuration save (real restart, real file, MariaDB back
up clean) and the Require-2FA toggle — which, turned on against this session's own non-2FA admin
account, immediately locked that session out of everything except its own Profile page, including
the settings page needed to turn the policy back off. That's not a bug, it's the feature actually
requiring what it says it requires — confirmed by resetting the setting directly rather than through
a UI the policy had just correctly locked.
