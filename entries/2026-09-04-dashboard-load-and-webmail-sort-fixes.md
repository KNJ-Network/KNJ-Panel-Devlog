# Phase 196 - Waiting in Line, Twice

Two unrelated reports landed the same evening, and both turned out to be the same kind of mistake:
something that should have happened all at once was happening one step at a time, and nobody noticed
because each individual step was fast.

## The dashboard that was never actually broken

"There's always a delay when I click Dashboard" is a hard report to act on: nothing crashes and nothing
shows in a log. But the dashboard's service-status checks ran one after another, each waiting politely
for the last to finish. Nine short waits in a row add up to one long one, right at the top of the page
a customer sees on login.

The fix is not a bigger timeout or a smarter cache. Every check is started up front and the answers are
collected afterwards, so the cost drops from nine round trips to roughly one. A single combined status
call was rejected on purpose, because one stuck check there would take every answer down with it.
Starting them separately keeps each result independent while still running concurrently.

## The toggle that toggled nothing

The second report came with two screenshots of the same inbox, sorted ascending and descending, showing
the same emails in the same order. The cause lived a layer down, in the IMAP library: descending order
correctly decided which messages land on a page, but the headers were then handed back in whatever order
the server returned them, regardless of the request.

Sorting by sender or subject already re-sorted the fetched page on the application side. Date sorting
was the one exception, built on the assumption that a date would come back in the order asked for. It
didn't. Applying the same after-the-fact sort to date was the entire fix.

## Same shape, different names

Both were a version of the same assumption: that something built to run in sequence would still feel
instant at scale, or that a request handed to another system would come back exactly as asked.

Tested (2819/2819).
