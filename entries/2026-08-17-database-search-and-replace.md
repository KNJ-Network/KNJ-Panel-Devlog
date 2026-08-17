# Phase 91 - Database Search & Replace

Restore from cPanel Backup shipped two days earlier with an honest gap: if a customer's database
has their old domain baked into a settings table, such as a WordPress site URL, the import doesn't
rewrite it. That's the right call, since blindly rewriting guessed strings is the kind of
confident-but-wrong behavior this migration work has been trying to avoid, but it leaves the
customer without a way to fix it short of a manual SQL client. This closes that gap.

## Not folded into the cPanel-restore feature

A domain change matters for any migrated site, so this is a standalone tool under Databases. The
cPanel-restore results page links to it when a domain mismatch is detected, but it works on any
database the account owns.

## The isolation boundary is the same one the Database Browser uses

Every operation runs as the account's own granted database user, never the administrator, for the
same reason as the Database Browser's query runner: the database's grant system is the real
boundary, not just an application-level ownership check.

## The bug a naive version would ship with

WordPress and a lot of other software store structured data in PHP's serialized format, which
records the exact byte length of each string. A plain string replace works until the replacement
has a different length, at which point the value silently stops reading back correctly. Established
tools handle this by unserializing the value, replacing every string inside it, and reserializing.
This tool does the same, and goes further: a serialized value containing an object anywhere is left
untouched, since decoding it safely isn't guaranteed to reproduce the original bytes.

## Scope, deliberately

Only tables with a single-column primary key are changed. Others are reported as skipped rather than
guessed at, because a reliable row-by-row update needs a reliable way to address exactly one row.

## Live-verified against real data

A table shaped like typical WordPress options and posts tables, with plain strings, a serialized
array, and a row that should never match, confirmed all three cases: plain strings replaced, the
serialized array reserialized with a correct length and reading back cleanly, and the non-matching
row byte-for-byte untouched.

## Also fixed: a sidebar gap

Feedback showed there was no menu item for cPanel migration. Restore a cPanel Account had routes,
a controller, and a working page, but no sidebar entry. A review of the rest of the sidebar found
the same gap on Import Account. Both now live in a new "Migration" navigation group.
