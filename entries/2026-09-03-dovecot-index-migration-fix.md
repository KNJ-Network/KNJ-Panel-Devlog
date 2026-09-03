# Phase 190 - The File That Came From Somewhere Else

A filter rule did exactly what it was told to do — watch for a word in the subject line, move the
message somewhere specific — and the mailbox it belonged to broke immediately afterward. Not the
message. Not the folder it landed in. The whole mailbox, every folder at once, gone from view.

## A quiet passenger from the old server

When a mailbox moves onto this server — restored from an old host, or shifted between two of this
server's own machines — everything inside its folder gets carried across together. Almost everything
in there is real: the messages themselves, the rules someone wrote for sorting them. But mixed in
among that real content are a handful of files that were never meant to be content at all — private
bookkeeping the mail software keeps for itself, tracking things like "have I already delivered this
exact message before." Those files are supposed to stay behind. The software that reads them expects
to build its own copy fresh, the moment it needs one, on whichever machine it happens to be running
on.

One of those files came along anyway. It looked harmless sitting there — a mailbox doesn't inspect
its own internal bookkeeping on its way in, so nothing about the move itself failed or warned of
anything wrong. It simply waited, unread and untouched, until the one specific action that would
actually open it: a filter rule, firing for the first time since the move, doing the very thing it
was built to do.

## What "already open" doesn't mean

The bookkeeping file that arrived was built by a different mail system, on a different machine,
shaped in a way this server's own version doesn't use. When the filter rule tried to read it the way
it normally would, what it found didn't match what it expected to find there at all — not empty, not
missing, just the wrong shape entirely. That single failed read was enough to bring down folder
listing for the whole mailbox, because the one broken file sat squarely in the path everything else
needed to pass through first.

Clearing it out and letting the mail software recreate that bookkeeping fresh fixed the mailbox
immediately, on the spot, live, while it was still being watched. Nothing about the actual messages
or the folder they were filed into was ever at risk — that file was never where any of the real
content lived.

## Making sure it never gets packed again

Fixing this one mailbox on the spot solves the moment. It doesn't solve the source. Both of the
places a mailbox's contents get gathered up to move — restoring from an old host, and shifting
between this server's own two machines — pack up everything sitting in the folder without asking
which parts are real content and which parts are the previous machine's own private notes to itself.
Every one of those private files was always a risk waiting for the right trigger to expose it; this
one simply happened first.

The fix draws a clear, permanent line between the two: messages and the filter rules someone actually
wrote come along, every time, unchanged. Every piece of the mail software's own private bookkeeping —
by name, specifically, not by guesswork — gets left behind on purpose, so the destination always
builds a version that actually matches the machine it's running on.

Proven against a stand-in mailbox holding one of each kind of file side by side — real messages, a
real filter rule, and every private file this fix is meant to catch — the version that comes out the
other side keeps exactly the first two and none of the third.
