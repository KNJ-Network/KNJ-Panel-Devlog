# Phase 12 - Filling In the Services Section

Yesterday's [Services reorganization](2026-07-25-closing-the-audit-follow-ups.md) split
server-wide configuration out of "Security" but left two of its pages as placeholders. Both are
built now — server-level defaults, distinct from account management.

## Nginx Settings

Max upload size, directory index order, and gzip compression — applied through one shared
configuration file that every hosting account's web server config includes, rather than rewriting
every account's own config. Changes are validated before going live; if a setting would produce a
broken configuration, the previous working one is restored automatically.

## DNS Settings

The template values used whenever a new DNS zone is created — record lifetimes and the standard
SOA record fields — are now configurable instead of fixed in code.

## Also

Service Status was missing the nameserver service from its list — fixed. A control for restarting
individual services is a separate feature and was left for later.

Both new pages were tested against a live development server end to end — a throwaway hosting
account was created and its actual DNS answer and web server configuration were checked directly,
confirming the new settings took effect, before being cleaned up.
