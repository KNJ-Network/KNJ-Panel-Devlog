# Phase 279 - Scheduled Jobs You Can See

A scheduled job is the kind of thing nobody looks at until it stops working. The old Cron Jobs page was a table of
raw timing fields, with no way to tell whether a job had run, whether it had worked, or what it had printed.

## What changed

**A schedule you can read.** Pick "Every day", "Weekdays", "Every Sunday" or "1st of the month", choose a time, and the
page says back, in words, when the job will next run. Custom timing is still there for anyone who wants it, with the
same live preview, and a clear message when a value is out of range instead of a silent failure.

**Every run kept.** Each job shows when it last ran, whether it worked, how long it took, and what it printed, with the
last thirty runs kept as history. Run now does exactly what the schedule would do, so a job can be tested without
waiting for midnight. A job that overlaps its own previous run is skipped, and one that runs for over an hour is
stopped.

**A warning when one fails.** A failing job raises one alert, through the same email, chat and webhook channels as the
rest of the panel, and then stays quiet until it has worked once and failed again. A job that fails every minute does
not fill an inbox.

**Everything else that runs on the server.** A Built-in tab lists the panel's own scheduled tasks and the server's
timers in plain words, and a small timeline shows when things run today, with a warning if several of your own jobs are
piled up at the same moment.

## What testing found

The first version passed every automated test and then failed on a real server: the results of each run were being
recorded, but the panel was not allowed to read them, so the history stayed empty. It showed up within the first few
minutes of trying it on a real machine, and was fixed and re-checked the same day. It is a good reminder of why
each page gets a real run before it is called done.
