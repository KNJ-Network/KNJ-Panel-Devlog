# Phase 54 - Packages & Resellers, Closed Out

Five items on this section's roadmap, one of them ("Reseller accounts") already in progress. Started
by browsing trywhm.net's real Resellers tree directly to make sure ours covered the same ground —
then a routing audit found something worth acting on before building anything: a Reseller's real,
already-reachable capability surface in this codebase is exactly eight actions, not cPanel's roughly
forty-five. Everything else in the Controller area is already either behind admin-only middleware or
independently gated at the controller level. That narrowed "Reseller permission levels" down to
something that actually maps onto this system, instead of building a much larger ACL that mostly
wouldn't do anything.

Account reassignment turned out to already half-exist — a Reseller could already be reassigned
one account at a time. The only real gap was a bulk multi-select UI, mirroring the same detached-form
pattern List Accounts already used for bulk removal.

Reseller accounts got a proper detail page (usage totals across every account they manage), plus
something cPanel doesn't cleanly map onto: suspending a reseller's own panel login independently of
the accounts they manage, since a reseller here isn't guaranteed to own a hosting account at all.

Per-package feature control added six real, already-optional account-side capabilities — FTP, WebDAV,
App Installer, Cron, Backups, API Tokens — as package-level toggles, enforced both server-side and in
the account-side nav so a restricted owner never sees a dead link.

Reseller permission levels shipped as an eight-item checklist, `null` meaning the full default set so
every existing reseller kept working unchanged until an admin actually touched their permissions.
Live verification caught a real gap here: the sidebar nav item was correctly gated, but the List
Accounts page's own header button and every row-level action were not — found by curl-grepping for
"Create Account" after stripping that permission and seeing it was still there. Fixed the same day.

Account migration/transfer was the biggest of the five. Export turned out to need no new privileged
script action at all — the panel's own php-fpm process already has group read access to its own
backup directories, so bundling a completed backup's files plus a manifest into one portable
`.tar.gz` is just `tar`, run as the app. Import is the real new surface: upload, list the archive's
members with `tar -tzf` and reject anything that isn't an exact flat filename match for one of five
fixed shapes before ever extracting a byte, then a new `import-account-archive` provisioning action
that restores files (stripping the old account's top-level directory name, since import almost always
lands on a different username than it was exported under), mail and DNS (both domain-keyed, so they
restore unchanged), and the account's own default database (explicitly remapped from the old username
to the new one, rather than trying to recreate arbitrary extra databases with fresh credentials —
scoped out deliberately, the same as import only supporting a single domain per account).

Live verification found a real bug before anything shipped broken: the import controller was
dispatching its own `ProvisionAccountJob` inside a chain, on top of the one
`AccountProvisioningService::createAccount()` already dispatches internally — provisioning the
account twice, concurrently, and the second attempt's `useradd` failing fast against the user the
first attempt had just created. Fixed by dropping the redundant dispatch and leaning on
`ImportAccountJob`'s own "account must be Active" guard as the safety net. Re-verified with a full
round trip: a disposable account with a marker file and a marker database row, backed up, exported,
the original account deleted, the archive imported back under the same domain, and both markers
confirmed byte-for-byte identical on the freshly restored account.

One more thing came out of that same live check, filed separately rather than folded in: deleting an
account cascade-deletes its `backups` rows at the database level, which bypasses Eloquent entirely —
so neither a backup's raw files nor now its export cache ever get cleaned up when the account goes
away. Pre-existing gap in the original backup engine, not something this section introduced, but
worth fixing on its own.
