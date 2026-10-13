# Phase 260 - The Queue That Waited for a Backup Server

Two jobs were sitting in the queue for over an hour: a website deploy and an SSL certificate. Neither was
broken. They were waiting behind something else.

## Eight copies of the same upload

The panel had one background worker doing everything, one job at a time. Eight jobs ahead of those two
were uploading backups to the off-site backup server, and each took about thirteen minutes to give up.
The backup server was fine; a network problem meant every connection quietly went nowhere, and the upload code waited politely for a reply that was never coming.

There were eight because the retry sweep queued a fresh upload for every backup that had not made it
yet, on every sweep, without checking whether one was already waiting. Two accounts and the panel
database, over a few days, became a pile.

## Fix the pile, then fix the shape

The uploads are now queued once per backup, a backup that has just failed is left alone for two hours, and
a destination that cannot be reached is abandoned in seconds instead of minutes. The bigger change is the
shape: uploads run on a worker of their own, so a slow backup server can only ever delay backups.

Once that was done it was obvious the same could happen elsewhere. cPanel imports, malware scans, bulk
subscriber emails and system installs can each run for the best part of an hour on a busy host, so each of
those has its own worker too. The short jobs (certificates, DNS, deploys, mail) keep the default worker to
themselves. Adding another worker is now one small file.

The upgrade now picks up every worker it finds.
