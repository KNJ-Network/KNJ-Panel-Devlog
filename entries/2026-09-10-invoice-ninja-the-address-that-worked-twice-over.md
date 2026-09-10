# Phase 215 - The Address That Worked, Then Didn't

The last entry ended on the first click-through working. It shouldn't have ended there.

## What the first test actually proved

A login page loading is a real signal, but it can hide a smaller problem behind it. The page loaded, and
so did every script and stylesheet it asked for up front, which was what the earlier fix was built to
correct. That is a real result, but it is not the same claim as "this app works."

## The second file this app never mentioned by name

Everything the page asked for up front had its address corrected. Anything it might ask for later did
not, because those addresses aren't written anywhere a text fix can reach. They are decided when the
running app reaches for a part of itself it hasn't loaded yet, by code compiled elsewhere long before the
install existed.

It was found by asking the app to actually navigate and watching what it requested next: the same file
requested at two different addresses, one correct and one not.

## What doesn't get patched

A compiled file isn't a template with blanks to fill in. Fixing it properly means rebuilding it, which
needs tooling this install is designed to avoid. Patching it convincingly would only make it look fixed
without confirming it is.

So the app comes back out of the catalog for now. The parts that were right stay as they are, and it is
simply not offered as something that works, because beyond the first click it doesn't yet.
