# Phase 227 - The Report That Couldn't Say Zero

The comment above the code was unambiguous: a Trash folder that doesn't exist yet just reports
zero bytes, same as anything else with nothing in it. That was the intent, written down plainly,
right above the line that didn't do it.

## A safety net with a hole sized exactly wrong

This script runs every privileged action under a strict mode — stop immediately if anything fails,
and treat a failure anywhere in a pipeline as a failure of the whole pipeline. That's the right
default for a script that touches real customer data as root: better to halt loudly than keep
going past a step that didn't work. But strict mode doesn't know the difference between "this
failure means something is actually wrong" and "this failure just means the folder isn't there
yet, which is completely normal for a mailbox that opened yesterday." It stops for both, the same
way, every time.

Checking a Trash folder's size by asking the filesystem directly is fast and simple, right up until
the folder in question has literally never existed — at which point the same command that normally
prints a number instead exits unhappy, and the safety net built to catch real problems catches this
one too, indiscriminately.

## Found by a page going quiet

The mailbox that broke this wasn't old or heavily used. It was new — new enough that nothing had
ever been filed into Trash or Junk, which is the ordinary state of a mailbox on its first day, not
an edge case that needed hunting for. The very first account checked hit it immediately, which is
usually a sign the failure mode isn't rare at all — it's just rare for anyone to have looked yet.
And because the strict-mode stop happens mid-loop, it doesn't just fail to report the one folder
that's missing. It stops reporting everything after that point too, including mailboxes that were
never in question at all.

## Letting "normal" through without lowering the guard

The fix isn't loosening the safety net — it's telling it, in exactly the two spots that need it,
"a failure here is not a problem, it's the expected shape of a fresh mailbox." Everywhere else in
the script, the strict stop-on-failure behavior stays exactly as strict as it was. This wasn't a
case for weakening the whole net to let one fish through — it was for cutting one hole exactly the
size of the one case that was never actually a problem.

## What actually finished

A mailbox with nothing in Trash or Junk yet now reports that honestly — zero, not a crash. The
comment above the code was telling the truth about what should happen the whole time; the code
underneath it just needed to actually agree.
