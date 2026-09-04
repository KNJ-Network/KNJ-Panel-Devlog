# Phase 196 - Waiting in Line, Twice

Two unrelated reports landed the same evening, and both turned out to be the same kind of mistake
wearing a different coat: something that should have happened all at once was quietly happening one
step at a time instead, and nobody had noticed because each individual step was fast.

## The dashboard that was never actually broken

"There's always a delay when I click Dashboard" is a hard bug report to act on — nothing crashes,
nothing errors, nothing shows up in a log. It's just slow, sometimes, in a way that's easy to wave
off as a feeling rather than a fact. But `ServiceStatusService::statuses()` — already fixed once this
week for a genuine hang (Phase 180) — was still doing the same nine `systemctl is-active` checks the
exact same way that hang came from: one after another, each its own process fork and its own round
trip to systemd, each one waiting politely for the last one to finish before it could even start. Nine
short waits in a row don't look like a bug. They just add up to one long one, right at the top of the
page a customer sees the moment they log in.

The fix isn't a bigger timeout or a smarter cache — it's not queuing them at all. Every check gets
started up front, all nine processes running at once, and only then does the code sit down and collect
the answers. The wall-clock cost drops from nine round trips to roughly one. The tempting shortcut —
one single `systemctl is-active` call listing every unit at once — was rejected on purpose: that
collapses nine independent processes back into one, and one process that hangs now takes every unit's
answer down with it, exactly the failure Phase 180 had just finished ruling out. Starting them
separately and only then waiting on each one keeps that guarantee intact — a single stuck unit still
can't touch anyone else's result — while still running concurrently instead of standing in line.

## The toggle that toggled nothing

The second report came with proof: two screenshots of the same inbox, one sorted ascending, one
descending, and the same emails in the same order in both. Not a subtle bug — a control doing visibly
nothing.

The instinct was to look at KNJ Webmail's own code first, and that instinct was wrong. The actual bug
lived a layer down, inside the IMAP library itself. Asking for messages in descending order does
correctly decide *which* messages land on a given page — the pagination math is sound. But the library
then fetches those message headers through a call that hands results back in whatever order the mail
server's own connection happens to return them in, which turns out to be ascending by message ID,
completely regardless of what order was actually requested. The sort direction shaped the guest list
correctly. It just had no say over the seating chart.

KNJ Webmail had already half-solved this without fully realizing it — sorting by sender or subject
already re-sorted the fetched page on the application side, because relying on the library's own
ordering for text fields was never trustworthy to begin with. Date sorting was the one exception,
built on the assumption that a date, at least, would come back in the order it was asked for. It
didn't. Removing that one exception — applying the same after-the-fact sort to date that from and
subject already had — was the entire fix.

## Same shape, different names

Neither of these was really about systemctl or IMAP specifically. Both were a version of the same
mistake: trusting that something built to run in sequence would still feel instant once there was
enough of it, or trusting that a request handed off to someone else's system would come back exactly
the way it was asked for. Worth remembering the next time either assumption looks reasonable.

Tested (2819/2819).
