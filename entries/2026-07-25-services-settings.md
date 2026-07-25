# Phase 12 - Filling In the Services Section

Yesterday's [Services reorganization](2026-07-25-closing-the-audit-follow-ups.md) split
server-wide configuration out of "Security" but left two of its pages as placeholders. Both are
built now, matching what WHM calls "Tweak Settings" — server-level defaults, distinct from
account management.

## Nginx Settings

Max upload size, directory index order, and gzip compression — applied through one shared
configuration file every hosting account's web server config already includes, rather than
rewriting every account's own config individually. Changes are validated before going live; if a
setting would produce a broken configuration, the previous working one is restored automatically
and nothing on the server is affected.

## DNS Settings

The template values used whenever a new DNS zone is created — record lifetimes and the standard
SOA record fields — are now configurable instead of fixed in code, matching WHM's own DNS zone
template editor.

## Also

Service Status was missing the nameserver service from its list since it was added a couple of
phases back — fixed. WHM's separate "restart individual services" control panel is a distinct
feature from settings and is intentionally left for later, rather than requesting more server
access for a smaller add-on right after yesterday's larger changes.

Both new pages were tested against the real server end to end — a throwaway hosting account was
created and its actual live DNS answer and web server configuration were checked directly,
confirming the new settings took effect, before being cleaned up.
