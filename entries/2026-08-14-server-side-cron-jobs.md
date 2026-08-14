# Phase 79 - The Feature That Cost Nothing to Build

Next on the gap list was admin-side scheduled tasks. Per-account Cron Jobs has existed for a while,
letting account owners schedule recurring commands as their own system user. What was missing was
the same thing for the server itself: a one-off sysadmin job, such as a log rotation tweak or an
ad-hoc cleanup, that doesn't belong to any single hosting account.

## Reusing what already generalized itself

We went looking for what the account-side feature needed from the provisioning script, expecting to
write a new action. It wasn't needed: the existing action already validates the target user and
installs a job list for it, and nothing in it assumes the user belongs to a hosting account. The
rest followed the same shape as the account-side feature: a table for server-level jobs, a service
that regenerates the whole job list from the database on every write rather than editing in place,
and a controller behind the same admin-access check as every other server-infrastructure page.

## What live-verify caught this time

Nothing broke. The value of testing against the real server anyway showed in the details: the exact
job line landed where expected, the notification line appeared and disappeared correctly as the
notify-email field changed, and the in-app confirm modal fired on delete.
