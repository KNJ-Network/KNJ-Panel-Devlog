# Phase 18 - Ten Client-Panel Features, One Day

A big batch day on the account (client) side, closing out most of what was left on the
client-panel feature list in one push, plus a naming cleanup that's been overdue for a while.

## Controller / Account, named as their own thing

The two panel areas had been called by informal working names internally since early on — but this
is its own product, not a whitelabel of someone else's. Every reference in the interface itself now
says Controller and Account instead. The real logo replaced its placeholder around the same time.

## Fixed: orphaned mail storage

A gap first spotted a couple of phases back, fixed properly now: deleting a hosting account (or
just an addon domain on one) removed its database records and system user, but not its actual mail
storage on disk — since mail storage is organized by domain name, not by system username, the
normal user-removal step never touched it. Both deletion paths now clean up the mail directory
too. Verified through the real account-deletion flow, not a simulated call: created a real mailbox,
deleted the account for real, confirmed nothing was left behind.

## The batch itself

- **Disk Usage** — a breakdown of what's actually consuming space in the account, not just a
  single total.
- **Directory Privacy** — password-protect any folder under the account's web root.
- **Contact Information / Account Preferences** — the account owner's own contact and notification
  settings.
- **Default Address** — catch-all routing for mail sent to an address on the domain that doesn't
  exist.
- **IP Blocker** — an account-level deny list, independent of anything set server-wide.
- **Hotlink Protection** — stop other sites from embedding this account's images directly.
- **Leech Protection** — detects a Directory Privacy login being used from more places than makes
  sense (a shared or leaked password) and can suspend it automatically.
- **Optimize Website** — a gzip compression toggle for the account's own site.
- **Indexes** — control how a folder with no index file lists its contents.
- **Error Pages** — custom 400/401/403/404/500 pages per domain.

Each of these went in with its own real end-to-end check against the live server rather than being
batched through untested — Directory Privacy in particular needed a follow-up fix once real
testing showed its file browser wasn't correctly scoped to the account's own web root, caught and
closed out the same day rather than left as a known issue.

## What's next

Deploy from Git — flagged as a genuine differentiator worth pulling ahead of the rest of the
older feature-parity list, not just another item to check off. That's the next thing being built,
now that this batch has closed out most of what was left in this general parity pass.
