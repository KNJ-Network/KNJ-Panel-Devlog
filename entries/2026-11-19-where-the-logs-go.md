# Phase 310 - Where The Logs Go

Log Rotation controlled one kind of log, the websites', and said nothing about the rest, so there was no way to tell
where a server's disk was going. Looking at what actually sits in the log folders found two things nobody was managing. The
system journal, which records everything the server's services say, had no limit of its own beyond the system's default of
several gigabytes. And the panel's own logs were not rotated at all, so they only ever grew.

## What changed

The page now shows every kind of log on the server with its size, how it is rotated and how long it is kept, and lists what
needs attention: a nightly rotation that failed or stopped, a journal with no cap, a log that nothing rotates. The website log
settings keep their 32-day minimum, now with the reason beside them.

Every server is given a sensible cap on its journal, about two months of history, and the panel's own logs are rotated weekly.
Both are applied once on upgrade and left alone afterwards, and the journal's limits can be changed from the page.
