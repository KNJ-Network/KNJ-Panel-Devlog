# Phase 54 - Packages & Resellers, Closed Out

Five items on this section's roadmap, one of them ("Reseller accounts") already in progress.
Started with a routing audit that found a Reseller can reach only a small fraction of the
Controller area, with everything else behind admin-only checks. That narrowed "Reseller permission
levels" to something that maps onto this system, instead of building a larger access-control model
that mostly wouldn't do anything.

Account reassignment turned out to already half-exist; the real gap was a bulk multi-select UI,
mirroring the pattern List Accounts used for bulk removal.

Reseller accounts got a detail page (usage totals across the accounts they manage), plus the
ability to suspend a reseller's own panel login independently of the accounts they manage.

Per-package feature control added six optional account-side capabilities — FTP, WebDAV, App
Installer, Cron, Backups, API Tokens — as package-level toggles, enforced both server-side and in
the account-side navigation so a restricted owner never sees a dead link.

Reseller permission levels shipped as a checklist, with the default set unchanged so existing
resellers kept working until an admin changed their permissions. Live verification caught a gap:
the sidebar item was gated, but the List Accounts header button and row actions were not. Fixed the
same day.

Account migration/transfer was the biggest of the five. Export bundles a completed backup into one
portable archive. Import is the real new surface: the archive is strictly checked before anything is
extracted, then files, mail, DNS, and the account's default database are restored under the new
account. Importing a single domain per account is supported for now.

Live verification found a bug before anything shipped broken: the import flow was provisioning the
account twice, and the second attempt failed. Fixed by removing the redundant step. A full round
trip with marker data confirmed the restored account was identical.
