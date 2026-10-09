# Phase 248 - The Folder Name the Script Wouldn't Accept

A feature can pass every test it has and still never have worked once. This is the story of a backup feature that, in a real run on a freshly built test server, rejected its first real backup.

## Two programs, one folder name

Taking a backup involves two pieces of code that have to agree. The panel's application decides
where the backup goes and names the folder. A small helper script does the actual work, and it validates every instruction it receives before acting. That strictness is deliberate.

The script's expected folder format was written first. Later, a review added a short suffix to the folder name on the application side so that two simultaneous backups could not collide. A sensible change, with its own tests. Nobody updated the script's expected format.

## Why nothing noticed

Every test of the backup code replaces the script with a stand-in, because the real one needs a real server. The stand-in cheerfully accepts whatever it's given. So the application was tested
against a script that agreed with it, and the real script was never part of the conversation. On a
real server the first backup anyone clicked ended in "invalid backup path".

It surfaced the first time a real backup was attempted on a freshly built test server.

## What shipped

The script now accepts both formats, so older backups still restore. More usefully, a new test checks every value the application generates against the script's own rules, so the two sides are now tested against each other rather than a stand-in.
