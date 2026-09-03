# Phase 184 - Teaching the Restore Itself to Look

Every piece of tonight's investigation had been manual — reading files by hand, recognizing a pattern
because a human happened to notice something didn't look right. That's not a repeatable safeguard.
The last, most important step wasn't finding what was wrong with one customer's account. It was making
sure the next restore doesn't have to be found by hand at all.

## A scanner that already existed, and the gap it had

There's already a real malware scanner in this panel — a genuine antivirus engine, run on demand
against a whole account's files. It never flagged any of tonight's backdoors, and that's not a bug in
it. Signature-based antivirus works by recognizing *known* malicious code — a fingerprint of something
that's been seen and catalogued before. What was found tonight wasn't that. It was bespoke, written
for this one attack, with no eval, no base64, none of the usual markers a scanner is trained to look
for. The only reason it was caught at all was reading the actual logic line by line and recognizing
what it was *doing*, not what it looked like.

## Writing down the one thing worth checking for

That's not something worth trying to generalize into "catch anything suspicious" — a rule that broad
just trains an admin to ignore the tool the first time it cries wolf over something in a real
WordPress plugin. The honest, narrow version of this fix is: the exact combination of ingredients that
technique needed, and nothing else. Enumerate a handful of temp-style directories, pull data straight
out of an incoming request, and either delete a file or execute one — that specific combination has no
legitimate reason to exist in ordinary application code. Every one of those ingredients on its own is
completely mundane. Only all of them together, in the same file, means something.

Before trusting that rule at all, it got tested against the real evidence: a full, real, ~66,000-file
website codebase and 22 separate databases, the exact one this whole investigation came from. Zero
false alarms. That's the bar a rule like this has to clear before it's allowed to delete anything
automatically.

## Deciding what "found something" should actually do

A flagged file and a flagged database row don't deserve the same response. A single PHP file is a
clean, contained thing — if it's not restored, nothing else about the account is affected, so the
right move is simple: don't copy it over at all, log exactly what was skipped and why. A row buried
inside a real database dump, sitting among thousands of other legitimate rows, is a different kind of
problem entirely. Automatically deleting the wrong line out of a live SQL file, by pattern-matching
alone, is exactly the kind of "quick automated fix" that quietly corrupts something else nearby — the
honest answer there is to flag it clearly and leave the actual decision to a person, not to guess.

## Where it actually runs

The natural place for this is the moment right before a restore's privileged copy step — after
everything has already been unpacked and read, but before any of it is written onto a real, live
account. A file that gets removed at that point was simply never restored in the first place; there's
no cleanup step needed afterward, because it never arrived.

## Verifying it against the real thing, and the real risk of a false accusation

Direct, deliberate tests: a reconstructed version of the actual technique found tonight, flagged
correctly. A handful of completely ordinary uses of the very same ingredients — a legitimate cache
write, a normal form handler reading `$_POST`, a routine cache-file cleanup — left alone, every one of
them, because only the full combination trips the rule. Live-verified against the real deployed code
too: the same reconstructed backdoor flagged correctly, an ordinary file sitting right next to it left
untouched.

Tested (2724/2724).
