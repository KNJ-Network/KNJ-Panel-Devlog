# Phase 18 - Ten Client-Panel Features, One Day

A big batch day on the account (client) side, closing out most of what was left on the
client-panel feature list, plus a naming cleanup that had been overdue.

## Controller / Account, named as their own thing

The two panel areas had been called by informal working names internally since early on. Every
reference in the interface now says Controller and Account instead, and the real logo replaced its
placeholder around the same time.

## Fixed: orphaned mail storage

Deleting a hosting account (or an addon domain) removed its database records and system user, but
not its mail storage on disk, since mail is organized by domain rather than by system user. Both
deletion paths now clean up the mail directory too. Verified through the real account-deletion flow
on a development server.

## The batch itself

- **Disk Usage** — a breakdown of what's consuming space in the account, not just a single total.
- **Directory Privacy** — password-protect any folder under the account's web root.
- **Contact Information / Account Preferences** — the account owner's own contact and notification
  settings.
- **Default Address** — catch-all routing for mail sent to an address on the domain that doesn't
  exist.
- **IP Blocker** — an account-level deny list, independent of anything set server-wide.
- **Hotlink Protection** — stop other sites from embedding this account's images directly.
- **Leech Protection** — detects a Directory Privacy login being used from more places than makes
  sense and can suspend it automatically.
- **Optimize Website** — a gzip compression toggle for the account's own site.
- **Indexes** — control how a folder with no index file lists its contents.
- **Error Pages** — custom 400/401/403/404/500 pages per domain.

Each went in with its own end-to-end check against a development server. Directory Privacy needed a
follow-up fix once testing showed its file browser wasn't correctly scoped, and that was closed the
same day.
