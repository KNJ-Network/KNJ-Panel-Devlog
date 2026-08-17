# Phase 91 - Database Search & Replace

Restore from cPanel Backup shipped two days ago with an honest gap: if a customer's database has
their old domain baked into a settings table — a WordPress `siteurl`, for example — the import
doesn't rewrite it. That's the right call (guessing at which strings in a database are "the
domain" and blindly rewriting them is exactly the kind of confident-but-wrong behavior this whole
migration-tools effort has been trying to avoid), but it leaves the customer with no way to fix it
short of a manual SQL client. This closes that gap.

## Not folded into the cPanel-restore feature

A domain change matters for any migrated site, not only one that came from a cPanel backup — someone
who just moved their own domain, or imported content some other way, has the same problem. So this
is a standalone tool under Databases, not a step bolted onto the import flow. The cPanel-restore
results page links to it when a domain mismatch was detected, but it works standalone from any
database the account owns.

## The isolation boundary is the same one the Database Browser already uses

Every query — schema discovery, the dry-run count, the actual row fetch and update — runs through
the account's own granted MySQL user, never root. That's not a new pattern; it's exactly what the
Database Browser's own query runner does, for exactly the same reason: MySQL's grant system is the
real boundary here, not just an application-level ownership check. A bug in this feature's own
authorization code can't reach another account's database, because the connecting user was never
granted access to it in the first place.

## The bug a naive version of this would ship with

WordPress (and a lot of other software) stores structured data as PHP's own `serialize()` format —
a string like `s:26:"http://old-domain.com";` where `26` is the exact byte length of what follows.
A plain string replace on that data works fine right up until the replacement isn't the same byte
length as the original, at which point the length prefix is now lying about the string's real size
and the whole value silently stops unserializing correctly. This is a well-known trap — real tools
built for exactly this job (WP-CLI's search-replace, the Better Search Replace plugin) all handle it
the same way: unserialize the value, walk the resulting structure replacing every string leaf, then
reserialize it, letting PHP recompute the length prefixes correctly instead of hand-maintaining them.

One thing further than most of those tools go: if a serialized value contains a PHP object anywhere
(not just at the top level — nested inside an array counts too), it's left completely untouched. The
tool decodes objects with `allowed_classes: false` for safety, and a value read back that way isn't
guaranteed to reserialize to the exact same bytes it started as, even in the parts that were never
touched. Skipping it entirely is the correct default when "leave it alone" and "possibly corrupt an
unrelated part of the row" are the two options.

## Scope, deliberately

Only tables with a single-column primary key get touched — anything with no primary key or a
composite one is reported as skipped, not guessed at, because a confident row-by-row UPDATE needs a
reliable way to address exactly one row. Every value that gets written goes through as a hex literal
rather than a quoted SQL string, which sidesteps escaping the search/replace text entirely — there's
no string-literal-injection surface to think about because there's no string literal.

## Live-verified against real data

A real MySQL table shaped like `wp_options`/`wp_posts` — plain-string columns, a serialized-array
column, and one row that should never match — confirmed all three cases: the plain strings replaced
correctly, the serialized array reserialized with a correct updated length prefix and unserializes
cleanly afterward, and the row that shouldn't have matched was still byte-for-byte untouched at the
end.

## Also fixed this session: a sidebar gap, found the same way the last one was

A user question ("I don't see a menu item for cPanel migration") turned up the exact same bug
Restore a cPanel Account itself shipped with two days ago: it had routes, a controller, and a
working page, but nobody had ever added it to the sidebar — only reachable via a link buried on the
Backups page. A quick audit of the rest of the sidebar turned up the identical problem on Import
Account (the older KNJ-Panel-to-KNJ-Panel migration tool). Both now live in a new "Migration" nav
group — which is also exactly where the still-open Transfer Tool and Convert Addon Domain to Account
work will land once it's built.

## Next

Transfer Tool (remote SSH pull from a live cPanel/WHM server) and Convert Addon Domain to Account —
the two still-open pieces of the original migration-tools roadmap line.
