# Phase 183 - The Fix That Wasn't the Whole Fix

The dashboard hang from earlier was supposed to be closed, but a satellite server still hung at login. The first fix was real and necessary, but not the only cause.

## Narrowing it down

The new version was confirmed deployed, and every backend step rendered in under a second. The browser reported "connecting", not "waiting", which pointed at a destination that could not be reached.

The page's scripts and stylesheets pointed at a different port than the page itself, one that this kind of server does not expose. A fix from the previous day had been written as a rule for the whole server, when it should have applied only to the shortcut routes it was designed for.

## The fix

The decision is now made per request, based on how the request arrived, so ordinary requests find their own assets. Already-running servers override the old setting early, and a routine update cleans it up.

## Verifying

An ordinary login page finds its assets, a request via a shortcut name is still handled correctly, and a server without the extra port no longer looks for it.

Tested (2714/2714).
