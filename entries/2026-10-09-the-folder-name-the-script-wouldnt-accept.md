# Phase 248 - The Folder Name the Script Wouldn't Accept

A feature can pass every test it has and still never have worked once. This is the story of a backup
system that, as far as anyone could tell, had been quietly rejecting every real backup for six weeks.

## Two programs, one folder name

Taking a backup involves two pieces of code that have to agree. The panel's application decides
where the backup goes and names the folder. A small, privileged script — the only part allowed to read
a customer's files and databases — does the actual work, and it checks every instruction it receives
against a strict pattern before touching anything. That strictness is deliberate; it's what stops a
malformed or malicious request from ever reaching the disk.

The script's pattern for a backup folder was written in July: an account name, a slash, and a
timestamp. In late August a concurrency review added a short random suffix to the folder name on the
application side, so that two backups started in the same second could never be handed the same
folder. A sensible change, with its own tests. Nobody touched the script's pattern.

## Why nothing noticed

Every test of the backup code replaces the script with a stand-in, because the real one needs root and
a real server. The stand-in cheerfully accepts whatever it's given. So the application was tested
against a script that agreed with it, and the real script was never part of the conversation. On a
real server the first backup anyone clicked ended in "invalid backup path".

It surfaced the day the first real backup was attempted after rebuilding the backup settings — a
week-old feature on a freshly built backup server, and the very first click of "Backup Now".

## What shipped

The script now accepts both shapes, the new one and the old, so backups made before the suffix still
restore. More usefully, a new test reads the script's own patterns straight out of the script file
and checks every value the application generates for a backup or a restore — the base folder, the
folder name, the temporary manifest file, the restore folder — against them. Putting the old pattern
back makes the test fail with the exact error from the live server. The two sides can't disagree
silently again.
