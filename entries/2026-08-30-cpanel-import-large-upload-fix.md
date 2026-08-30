# Phase 166 - The 4GB Limit That Was Never Real

A real migration, finally: a genuine cPanel full-account backup, ~975MB, exported from a real hosted
site being moved onto this stack. First real-world test of "Migrate a cPanel Account" against
anything bigger than the synthetic archives it shipped against back in Phase — and it failed
immediately. Firefox reported a connection reset partway through the upload; the WHM page itself gave
no indication anything had gone wrong beyond a spinning browser tab.

## Chasing the actual limit

The nginx error log on the live server told the real story: `client intended to send too large body:
1022026100 bytes`, repeating every ~30 seconds as the browser silently retried the same doomed upload
into the same wall. Something was capping the request well below the size of the file — the question
was what, and where.

The feature's own advertised ceiling is exactly 4GB (`CpanelImportService::MAX_ARCHIVE_BYTES`,
`max:4194304` on both the WHM and account-side validation, `"4 GB limit."` on both views — three
independent hardcodings, all consistent with each other). Nothing in the actual server configuration
came close. `client_max_body_size` — the customer-facing 100MB default, admin-configurable via WHM →
Nginx Settings — turned out not to be the culprit at all: that snippet is deliberately included only
in *customer account* vhosts, never the panel's own controller (2087) or account (2083) vhosts,
specifically so a WAF false positive on that snippet can never lock an admin out of the panel's own
login screen. Neither `bootstrap-server.sh` (fresh installs) nor `knjpanel-provision-account`'s
`issue-panel-cert` action (the vhost regeneration path used on every hostname/cert change) ever set
`client_max_body_size` for the panel's own service at all — meaning the real limit in effect wasn't
100MB, it was nginx's compiled-in 1MB default, silently inherited by omission the entire time this
feature has existed.

PHP told the same story one layer down. `PhpSettingsService` already generates a shared php.ini
override covering every PHP-FPM pool on the box, including the panel's own — but `bootstrap-server.sh`
never seeds it at install time, so it only exists once an admin visits WHM → PHP Settings and saves.
Nobody ever had. Live `php.ini` on the production server was still stock Ubuntu/ondrej defaults:
`upload_max_filesize = 2M`, `post_max_size = 8M`. And even if an admin *had* gone looking to fix it
by hand, they'd have hit a second wall: `PhpSettingsController`'s own validation capped both fields at
2048MB — half the feature's own advertised limit, unreachable no matter what was typed in.

## The part that turned out to already be fine

Before touching anything, it was worth checking whether `max_execution_time` (30s stock) was also
going to be a problem for the actual tar extraction — a multi-hundred-MB archive doesn't unpack in 30
seconds. It didn't need to be: `ImportCpanelAccountJob`/`ImportCpanelIntoAccountJob` already dispatch
the real extraction/restore work as a queued job (`$timeout = 1830`), same pattern this project has
used for every other slow-and-synchronous risk since the Perl/PEAR/PHP-extension conversions back in
Phase 106-ish. The web request only ever needed to survive receiving the upload and a quick staging
check — the actual restore was never going to be killed by a short PHP timeout, because it was never
running inside PHP's request lifecycle to begin with.

## The fix

Three changes, kept to exactly the surfaces that needed them:

- `client_max_body_size 4300m` (plus matching `client_body_timeout`/`fastcgi_read_timeout`, so a slow
  upload isn't cut off mid-transfer either) added to the panel's own 2087/2083 vhost blocks in both
  places that generate them — `bootstrap-server.sh` for new installs, `knjpanel-provision-account`'s
  `issue-panel-cert` action for existing ones. Confirmed that action rewrites the vhost unconditionally
  even on a no-op hostname reconfirm, which matters: it means an already-installed server can pick this
  up just by an admin hitting Save on the existing Server Setup page, no server access required.
- `PhpSettingsController`'s validation ceiling raised from 2048 to 8192 — the underlying setting stays
  admin-opt-in (this ini override is shared by every PHP-FPM pool on the box, hosting accounts
  included, so the *default* wasn't touched — only the artificial cap preventing an admin from
  choosing a real value).
- Left everything about the queued-job extraction path untouched — it was already correctly scoped.

## Verifying past the code

Full suite (2563/2563) and `pint --test` first — the usual gate before anything ships. The real
verification is still pending: this fix goes out through the normal release pipeline (dev → panel-dev
autodeploy → version bump → release cut → all 4 production servers → live-verify) before the actual
975MB migration gets retried for real. The archive itself was confirmed intact and well-formed while
diagnosing this (`tar -tzf` reads clean end to end, 69k files across 5 domains) — the failure really
was purely infrastructure, not a corrupted upload.
