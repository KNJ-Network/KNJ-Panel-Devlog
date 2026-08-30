# Phase 163 - A Report Nobody Has to Remember to Check

Subscribers had real-time stats, but nothing pushed activity to the owner. Feedback asked for a daily email, sent to a chosen address at a chosen time, summarising a list's activity.

## Per-list scheduling

The scheduler is built at boot, so per-list times cannot be registered individually. Instead, the report job runs on a short regular tick and each list's own stored time is compared with the current time, with a last-sent date preventing repeats and letting a delayed tick catch up.

## Time and content

There is no per-account timezone, so the report time is server time and the UI says so. The report covers a rolling 24-hour window: new subscribers, unsubscribes, pending confirmations and the current total. An option skips sending when there is no new activity.

## Verifying

Local tests use frozen time. On the development server a disposable list produced a real report through the queue and mail pipeline. Tested (2561/2561).
