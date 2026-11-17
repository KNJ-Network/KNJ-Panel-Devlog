# Phase 290 - Do The Backup Servers Work?

DNS Clustering exists so that if the main DNS server goes down, a second one keeps answering for every domain. The page that
managed it showed a switch and a list of server names, addresses and keys. It could not say whether a backup server was on,
whether it held any zones, whether those zones were current, or whether any domain's nameservers even included it, which is
the only way a backup ever gets asked a question. Every server's secret key was also sitting on the screen permanently, and
the switch turned off with a single click and no explanation, even though turning it off quietly starts a countdown after
which the backups stop answering altogether.

## What changed

The page now checks. Each zone carries a serial number that rises whenever the zone changes, so asking the main server and
each backup directly for that number says whether the copy is current, behind, or absent. The overview shows how many
copies are up to date, whether each backup is online, and whether it is actually used as a nameserver, and flags a backup
that holds every zone but that nobody is pointed at. Each backup has its own page listing every zone with both numbers,
problems first, with a button to send it the zone list again and another to replace its key. The key itself is hidden until
you ask to see it.

Turning clustering off now says what it means: the backups keep answering from what they have, miss every change from then
on, and stop answering after the usual waiting period. A backup that falls behind or goes quiet for two checks in a row is
reported to the administrators once, instead of nobody finding out until the day they are needed.
