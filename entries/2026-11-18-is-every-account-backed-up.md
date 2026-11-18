# Phase 302 - Is Every Account Backed Up

The Backups page was a history table with a button for every account, which meant hundreds of buttons on a big server, and it
could not answer the one question that matters: is everything backed up? A failed backup showed a raw shell error, an account
left out of the schedule looked exactly like one that was included, and a failure that happened weeks ago stayed on the page
for ever. Settings, destinations and account selection were three more pages.

## What changed

The page now starts with the answer: how many accounts have a good backup inside the schedule, when the last good one finished,
where the copies are and when the next run is. A list at the top puts what needs attention first, each with the reason in plain
words and a button: an account whose backup is failing, one that has never been backed up, a destination that cannot be reached,
or nothing leaving the server at all.

The full list of backups is paged and filterable, and every account has one row showing its newest good backup, where it is
kept and how many restore points it has, with a single switch for whether it is in the nightly run. Settings and destinations
sit beside them as tabs.

A backup started by hand now runs in the background, so the page returns at once and a large account can no longer time the
page out. One that cannot even start still leaves a visible entry saying why. Administrators are also told when a single account
has gone without a good backup while the rest are fine, which used to go unnoticed as long as anything else completed.
