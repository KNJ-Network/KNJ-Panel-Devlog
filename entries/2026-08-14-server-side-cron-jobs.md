# Phase 79 - The Feature That Cost Nothing to Build

Next on the gap list: admin-side scheduled tasks. Per-account Cron Jobs has existed for a while —
account owners can already schedule their own recurring commands, run as their own system user.
What was missing was the same thing for the server itself: a one-off sysadmin job (a log rotation
tweak, an ad-hoc cleanup script) that doesn't belong to any single hosting account.

## Reusing what already generalized itself

Went looking for what the account-side feature actually needed from the privileged provisioning
script, expecting to write a new action. It wasn't there to write — `cron-sync` already takes a
system username and a temp file path as its two arguments, validates the username against the
same general pattern every other username goes through, and hands both straight to
`crontab -u`. Nothing in it assumes the username belongs to a real hosting account. Pointed it at
`root` instead of an account's own uid, and it worked on the first real test.

The rest followed the same shape as the account-side feature almost exactly: a `system_cron_jobs`
table (no `account_id`, since these jobs don't belong to one), a service that regenerates root's
whole crontab from the database on every write rather than editing it in place, and a controller
gated by the same `admin.access` middleware every other server-infrastructure page in the
Controller already sits behind.

## What "live-verify" caught this time

Nothing broke — this one built clean start to finish. The value of testing against the real server
anyway showed up in the details rather than a bug: confirmed the exact cron line landed in root's
actual crontab via `crontab -l -u root`, confirmed the `MAILTO=` line appeared and disappeared
correctly as the notify-email field changed, and confirmed the existing in-app confirm modal
(no native browser `confirm()`, panel-wide) fired correctly on delete. All from a page that reused
existing plumbing end to end.

## Next

Continuing down the gap list — Awstats or Backup User Selection next.
