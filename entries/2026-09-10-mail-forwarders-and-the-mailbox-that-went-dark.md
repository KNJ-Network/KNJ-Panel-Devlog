# Phase 219 - The Mailbox That Went Dark

An address can have two things pointed at it: a real inbox, and a rule that says "also send this
somewhere else." Nothing about those two ideas conflicts, but the underlying mail configuration had one
lever, and using it replaced rather than added. This was found in our own testing.

## A feature with a silent opposite

Creating a forwarder for an address that already had a mailbox didn't fail or warn. It succeeded, and
from then on the mailbox stopped receiving mail, with nothing in the panel saying so. The forwarder was
doing what a forwarder does; at that layer of the mail system, "redirect this" and "replace this" are the
same instruction. Nothing was built to notice when someone asked for both.

## Two ways to add "and", only one of them safe

The obvious fix is to let one rule point at two destinations. That sounds like a formatting detail, but
it changes behavior at the layer that decides whether mail is delivered at all, in a way that is hard to
verify, and for every address rather than the one being tested.

The safer way is to leave that layer alone. Mail is delivered once, normally, and the mailbox's own
local rules make a copy on the way in, the same mechanism that already files messages into folders or
replies automatically. The address is no longer treated as a redirect, and everything about "also
forward this" happens downstream, affecting only the one mailbox it is attached to.

## What refusing to guess cost

It was slower. It meant reading how a message already gets filed, confirming the delivery mechanism has
an explicit "keep going after this" mode, and extending that rather than taking a one-line shortcut at
the layer with the most to lose.

## What a checkbox is standing in for

What looks like an "also keep a copy" checkbox stands in for two refusals: a forwarder can no longer be
created at an address that already has a mailbox, and a mailbox can no longer be created where a
forwarder is already active. Both used to succeed wrongly. Now they ask once, before anything goes dark.
