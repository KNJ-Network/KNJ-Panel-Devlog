# Phase 249 - One Clock, Not Two

"Server time" is one of those phrases everyone is sure they understand. A backup was set to start at
02:00, the settings page said "server time (UTC)", and the server's own clock said London.

## The clock the panel was reading

The operating system had been told to use London time — the timezone chosen on the Server Setup page,
which describes itself as the server's timezone. But the panel's own application runs on UTC, on
purpose: every timestamp already stored in its database is UTC, and changing that clock would quietly
shift all of it. So when an admin typed 02:00, the panel read it as 02:00 UTC, which in British summer
time is 03:00 on the wall. The backup page also announced the next run at the current time rather than
02:00, because enabling the schedule mid-morning meant "today's start time has already passed" — and
the rule was "run if it's past the start time and hasn't run today". Switching backups on at 10:35
would have started a full backup of every account a few minutes later.

## One source of truth

The timezone now has exactly one owner: whatever the operating system is set to, read once and cached
briefly. Everything else asks it.

- **Schedules** read the admin's time in that zone — backups, malware scans, subscriber reports — and
  the scheduler's own fixed jobs run in it too. The once-a-day guards use the server's date, so a run
  at 00:30 isn't mistaken for yesterday's.
- **Display** goes through a single conversion applied wherever the panel shows a moment in time:
  backup history, the action log, certificate expiry dates, deploy history, mail delivery, the
  webmail inbox's "today" and "yesterday". Stored data is untouched; only how it's read and shown.
- **Edges**: a start time missed by a few minutes (a restart, a busy server) is still caught for up to
  three hours, then skipped until the next scheduled day, and saving a schedule after today's start
  time no longer triggers a run. Changing the timezone takes effect immediately.

Every field where an admin types a time now names the zone next to it, and links to where it's set.
