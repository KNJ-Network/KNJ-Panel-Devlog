# Phase 251 - The Run That Had Already Happened

The first real test of the new backup schedule failed before it started — not because anything broke,
but because the system was certain it had already done the job.

## A promise made at the wrong time

Scheduled backups are guarded so they run once a day and never twice: when a run starts, today's date
is written down, and the scheduler skips any day that is already written down. Sensible. The trouble
came from how that date got written, earlier the same morning: switching the schedule on at ten-thirty,
under an older build, started a full backup straight away — the start time had already passed, so the
rule fired immediately — and the system dutifully marked today as done.

Later, with the fixed build running, the test was simple: set the start time for a few minutes ahead and
watch it fire. The settings page said the next run was tomorrow. Today's date was already written down,
left over from a start time that no longer existed.

## Saving means "line today up"

Saving the schedule now reconciles today with whatever was just chosen. If the new start time has already
passed today, today counts as done, so saving at ten-thirty for two in the morning waits for tomorrow, as
before. If the new start time is still ahead, any note saying today already ran is cleared, because it
belongs to a different schedule — and the run happens at the new time. Weekly and monthly schedules do
this only on their scheduled day.

The same day's real checks on the live server were reassuring: a manual backup of a real account and of
the panel database both completed, each file was uploaded to the backup server and verified, the copy on
the web server was removed automatically, and the backup server's storage held the complete set — files,
every database, DNS zones, mail and FTP accounts — with the web server's backup folder empty.
