# Phase 277 - Logs That Tell You What Matters

A server on the internet is probed all day, so a log full of errors usually means very little. The Logs page showed the
last two hundred lines as one block of text, which made it impossible to tell the routine from the real.

## What changed

**A verdict first.** Each log opens with a sentence: "Mostly noise, with one thing worth a look". It separates the
background (scanners asking for files that should never be public, bots trying to log in, strangers trying to relay
mail) from the things that are actually yours to deal with.

**Repeats grouped and explained.** Twenty-two identical lines become one line with a count, a small chart of when it
happened, and a plain note on what that kind of message usually means and what to do about it. A message it does not
recognise is never called harmless: an unexplained error is marked for a look.

**And everything you would expect to find.** Search that highlights as you type, a filter for errors and warnings,
up to a thousand lines, a mode that follows the log live, download, and a badge on each tab showing which logs have
something worth looking at before you open them.

## What reading the logs found

Before building any of it we read the logs of our own busiest server by hand, to see what the page would have to deal
with. Almost everything in them was noise, as expected. Two things were not. A
nightly backup of one busy site had failed twice in two weeks because the site changed a file while it was being
archived, and the backup tool treated that harmless warning as a failure. It now treats it as a warning, and a real
problem still fails the backup. And one background job had been repeating the same error for weeks, which was fixed in
the same release.
