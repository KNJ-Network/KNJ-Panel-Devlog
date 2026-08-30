# Phase 163 - A Report Nobody Has to Remember to Check

The Subscribers feature shipped six days ago with real-time stat cards on every campaign, but
nothing that pushed activity *to* anyone — an owner had to remember to open the panel and look.
Requested directly: a daily email, sent wherever they want, at whatever time they want, telling
them what happened on a list since yesterday.

## Why this couldn't just be another `dailyAt()` line

`bootstrap/app.php` already has exactly one admin-configurable daily job — `RunBackups`, pinned to a
hardcoded `02:00`. That's server time, chosen once, and nobody expects to change it per account. This
feature needed the opposite: every list on the server potentially wants a different time, chosen by
whoever owns it, through a form. Laravel's scheduler is built at boot time from a fixed set of
registration calls — there's no way to say "run this at whatever time is sitting in row #4,382 of a
database table" using `->dailyAt()` directly.

The fix already had a name, just never applied to a *per-row* schedule before: `BackupService`'s own
admin-configurable cadence doesn't touch the registration line either — `RunBackups` still just runs
`dailyAt('02:00')`, and the actual "is today really a backup day" decision happens inside
`BackupService::runScheduled()` itself, read fresh off `Setting::get('backups.schedule')` every time.
`SendSubscriberListReports` copies that shape one level further: it registers as a plain
`everyFiveMinutes()` tick — matching `CheckServerHealth`/`CheckWafHealth`'s own cadence for this
class of "notice soon, not instantly" check — and `SubscriberListReportService::runScheduled()` does
the actual per-list decision, comparing each enabled list's stored `report_time` (`'HH:MM'`) against
the current minute-of-day.

The obvious risk with "checked every five minutes, fired once threshold crosses" is firing on every
tick for the rest of the day. Guarded the same way `AccountNotificationService` guards its
disk-usage and SSL-expiry checks: a `report_last_sent_date` column, stamped the moment a report goes
out, checked first on every run. A list whose scheduled minute has already passed *and* who already
has today's date stamped is a no-op — cheap, and it means a slow tick or a five-minute window
catches up rather than silently skipping the day if the worker was briefly down when the exact
minute passed.

## No timezone to hang this on

Went looking for a per-account timezone setting before writing the comparison logic, on the
assumption a "9am" input should mean the account owner's 9am. There isn't one, anywhere in this
codebase — `config/app.php` pins `UTC` app-wide, and the only "timezone" concept that exists at all
is the *server's own OS clock*, changeable only by an admin via `timedatectl` on the Server Setup
page, which is a machine-wide setting with nothing to do with any individual account. Introducing a
real per-account timezone concept just for this one form field would have been a meaningfully bigger
feature than the one actually requested. `report_time` is server time, stated plainly in the UI next
to the picker — same honesty `RunBackups`' own hardcoded `02:00` already gets away with, just now
admin-chosen instead of code-chosen.

## The report itself, and the one deliberate non-choice

`SubscriberListDailyReport` is a `Notification` + markdown `MailMessage`, not a `Mailable` — the
existing precedent for "system-generated, no admin-authored content" mail in this codebase
(`DiskUsageWarning`, `SslExpirationWarning`), as opposed to the raw-HTML `Mailable` pattern welcome
emails and campaigns use, where the account owner wrote the body themselves. Sent via
`Notification::route('mail', $list->report_email)`, the same on-demand-routing shape
`AccountNotificationService` already uses for arbitrary destination addresses that aren't tied to a
`User` model — the whole point of the destination field being a plain text input rather than a
mailbox picker is that it doesn't have to be an address on the account at all.

Stats are a rolling 24-hour window ending at send time — new subscribers, unsubscribes, pending
double-opt-in confirmations, and the current total — computed fresh on every send rather than
carried forward from the last report, so a list that goes quiet for a few days and then gets a
report still describes exactly what happened in the last real day, not some average across the gap.

Added one more setting beyond what was explicitly asked for: "only send when there's new activity."
A report that says "0 new, 0 unsubscribed" every single day for a quiet list is exactly the kind of
notification people learn to stop reading. Checked, it still stamps `report_last_sent_date` (so it
doesn't recompute stats on every tick for the rest of the day) — it just skips the actual send.

## Verifying it against a real queue worker, not a fake

Local suite covers the scheduling logic itself with frozen time (`Carbon::setTestNow`) — fires once
the threshold passes, doesn't resend same-day, does resend the next day, respects the skip-if-empty
guard, and correctly excludes disabled lists or ones missing a destination address. What that
can't prove is whether the real pipeline — queue worker, `Notification`, Postfix — actually holds
together end to end.

Created a disposable list on `panel-dev` with `report_time` set two minutes in the past and one real
subscriber added in the last hour, then ran `knjpanel:send-subscriber-list-reports` directly.
`journalctl` on the queue worker showed `App\Notifications\SubscriberListDailyReport ....
RUNNING` and, 244ms later, `DONE`, and Postfix's own log showed a real delivery attempt for the
message that job produced — bounced only because the destination address was a mailbox that doesn't
exist on that domain, a fact about the disposable test address, not the feature. `report_last_sent_date`
confirmed stamped afterward. Deleted the test list.

Tested (2561/2561).
