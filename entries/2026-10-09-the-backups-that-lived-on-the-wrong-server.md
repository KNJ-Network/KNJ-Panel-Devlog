# Phase 247 - The Backups That Lived on the Wrong Server

A backup server exists so that backups aren't stored on the web server. The panel had been
quietly doing the opposite.

## Mirror, not move

Every backup was built on the web server's disk and then copied to any configured destination as a
"best-effort mirror" — and then kept, for the full retention period, in both places. Restores and
downloads read only the local copy. There was no setting to say "once it's safely stored elsewhere,
free the space here", because nothing in the panel actually knew whether a destination held a
backup. Uploads weren't tracked; a failed one was a line in a log.

Fixing that properly meant giving the system a memory first.

## Knowing before deleting

Each backup now records, per destination, whether a verified copy exists — verified meaning every
file's size on the destination matches the file it came from. The new "move it off this server"
choice removes the local copy only once every enabled destination holds a verified copy. A failed
upload keeps the local files and is retried on every scheduled run, with an alert. With no
destination enabled, nothing is removed. A mutation check — deliberately deleting that rule and
confirming a test fails — proved the safety isn't just asserted.

## Getting a backup back

A backup stored only off the server stays in the history list. Restoring or downloading it starts
with **Fetch** (or **Bring back** for the account owner): a background job pulls the files into a
staging folder, checks each one against the size recorded at upload, and only then moves the folder
into place, so a half-fetched backup can never look restorable. The fetched copy is cleaned up again
after a configurable number of hours. Names coming back from a remote system are treated as
untrusted and never allowed to escape the folder.

## Two kinds of retention

The server keeps its copy for N days; each destination can keep its own copies for its own N days,
and the panel deletes expired ones from it. A sweep also notices when a destination expired a backup
itself (a bucket's own lifecycle rule) and drops the stale record — while treating an unreachable
destination as "unknown", never as "everything is gone". Deleting a backup now removes it
everywhere, and the record survives if a destination was unreachable so it can be retried.

## A settings page that had four options

The old settings page had a checkbox, a path, a frequency and a number. Now: a start time, weekday
and month-day, an option to leave the panel database out, the local-copy choice, how many of the
newest backups always stay local, a free-space safety stop that refuses to start a backup that
would fill the disk, and how long fetched backups stay — with a live strip showing the next run,
free space and what's stored where. The history page shows, per backup, where it lives. The
scheduler wakes every fifteen minutes and starts the run once, after the chosen time.
