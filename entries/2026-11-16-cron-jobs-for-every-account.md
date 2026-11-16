# Phase 280 - Cron Jobs for Every Account

The server-side Cron Jobs page was rebuilt in the last phase. This phase gives the same page to every hosting account.
Until now a customer's scheduled jobs were a table of raw timing fields with no way to edit one, and any output was
emailed on every run, which most people learn to ignore.

## What changed

**Schedules in words.** Choose "Weekdays at 06:30" or "Every 15 minutes" and the page says when the job will run next.
Custom timing is still there, and a value that is out of range is explained instead of silently refused.

**Edit, switch off, run now.** A job can finally be changed without deleting and re-adding it, paused without losing it,
and started on the spot to see whether it works. Suggested starting points cover the usual cases: a WordPress site's
scheduled tasks, a PHP script, calling a web address, tidying old files.

**Every run kept.** Each job shows when it last ran, how long it took, its exit status and everything it printed, with
the last thirty runs kept. A job that overlaps its own previous run is skipped, and one that runs for over an hour is
stopped.

**One email when it matters.** Instead of mailing the output of every run, the account owner is emailed once when a job
starts failing, with the last lines it printed, and not again until it has worked and failed again.

## Staying inside the account

A customer's job runs as that customer and nothing else, and the rebuild keeps it that way. The server never reads or
writes a customer's files as itself for this feature: everything that touches an account's jobs happens as the account's
own user, and what comes back is checked again before it is believed. We tested it by trying to break it from inside an
account: a result forged for somebody else's job, a link pointed at a file it should not read, an attempt to make the runner
run something it should not. None of them got anywhere.

Accounts that already had jobs are moved over automatically, a few at a time, with nothing to do.
