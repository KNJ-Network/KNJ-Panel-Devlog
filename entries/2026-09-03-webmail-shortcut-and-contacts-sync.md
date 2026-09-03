# Phase 186 - The Shortcut That Didn't Ask

A restored account's contacts had already been confirmed safe once — verified over SSH, matched
against real family names, real email addresses, nothing missing. Then the actual owner tried to use
them, and it turned out that verification had answered the wrong question.

## Checking the wrong thing correctly

The contacts genuinely had copied over — that part was true, and had been checked properly, directly
against the mail server's own database, without needing anyone's password. What hadn't been checked
was which of the two webmail interfaces the account owner actually uses day to day. His Email
Accounts page had that set explicitly. The contacts landed in the other one.

That surfaced a second, connected problem: the memorable shortcut — `webmail.<domain>`, the kind of
address a person actually types or bookmarks — didn't look at that setting at all. It always opened
the same interface, no matter what the account was configured for. There was already a correct answer
to "which interface should this account see" sitting one page over, on the "Launch Webmail" button.
The shortcut had simply never been taught to ask it.

## Making one decision in one place

The fix wasn't a new rule. It was finding the place that already made this decision correctly and
applying the same logic where it was missing — nothing invented, nothing to keep in sync between two
separate implementations of the same choice. A site configured for the other interface now sends the
shortcut straight there; a site left on the default is untouched, exactly as before.

## Two clients, one set of contacts, no shared code

The contacts gap needed something more than a routing fix, though — even pointed at the right
interface, an owner's contacts still only existed on one side. Fixing that meant keeping two
completely separate contact lists in sync, continuously, in both directions, without changing a line
of the other application's own code.

The way that's possible at all: the other interface's contacts aren't hidden behind its own private
logic — they're just rows in an ordinary database table, one any other program on the same server can
read and write directly. That's the exact same trick already used the day contacts were first taught
to survive a restore. This extends it from "once, during a restore" to "continuously, for as long as
the account exists" — read on every visit to the contacts page, written the moment anything changes,
either direction, whichever side the change was made on.

## The check that stopped a quiet bug before it shipped

Before any of that could be trusted, the actual shape of the other side's real data got checked
directly — not assumed, not copied from documentation, read straight from a real account's real
database. Good thing it was: the obvious way to write "update this contact if it exists, otherwise add
it" turns out to depend on a safeguard that database doesn't actually have. Written the obvious way, it
wouldn't have thrown an error. It would have quietly added a new row every single time anyone edited a
contact, forever, on every account using both interfaces — the kind of bug that never announces itself
and just keeps a database growing more wrong.

Rewritten as "remove the old entry, then add the new one" instead, it needs no such safeguard to exist
at all.

## Proving it against the real thing, not a stand-in

Everything got checked against an actual account's actual data on the real database, not a simulated
one: a contact added on one side lands correctly formatted on the other; editing it in place replaces
it cleanly, with nothing left duplicated behind; deleting it removes it the same way the other
interface already expects; and a contact added directly on the other side is picked up correctly too,
formatting included. The same checks were repeated a second way — through a linked secondary mail
server rather than the main one — since accounts don't all keep their mail in the same place, and
nothing about this was allowed to assume they do.

Tested (2752/2752).
