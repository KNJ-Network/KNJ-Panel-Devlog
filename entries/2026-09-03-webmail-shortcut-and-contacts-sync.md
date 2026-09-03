# Phase 186 - The Shortcut That Didn't Ask

A restored account's contacts had already been confirmed safe once, checked against the real contact
entries with nothing missing. Then the actual owner tried to use them, and it turned out that
verification had answered the wrong question.

## Checking the wrong thing correctly

The contacts genuinely had copied over. What hadn't been checked was which of the two webmail
interfaces the account actually uses day to day. The account's Email Accounts page had that set
explicitly. The contacts landed in the other one.

That surfaced a second, connected problem: the memorable shortcut, `webmail.<domain>`, the kind of
address a person types or bookmarks, didn't look at that setting at all. It always opened the same
interface. There was already a correct answer to "which interface should this account see" one page
over, on the "Launch Webmail" button. The shortcut had simply never been taught to ask it.

## Making one decision in one place

The fix wasn't a new rule. It was finding the place that already made this decision correctly and
applying the same logic where it was missing, so there is nothing to keep in sync between two
implementations of the same choice. A site configured for the other interface now sends the shortcut
straight there; a site left on the default is untouched.

## Two clients, one set of contacts

The contacts gap needed more than a routing fix: even pointed at the right interface, contacts still
only existed on one side. The two contact lists are now kept in sync in both directions, without
changing the other application's own code, using the same approach already used when contacts were
first taught to survive a restore, extended from "once, during a restore" to "continuously".

## The check that stopped a quiet bug before it shipped

Before trusting any of that, the real shape of the other side's data was checked directly rather than
assumed. Good thing: the obvious way to write "update this contact if it exists, otherwise add it"
depends on a safeguard that data store doesn't have, and would have quietly added a duplicate every
time a contact was edited. It was rewritten as "remove the old entry, then add the new one", which
needs no such safeguard.

## Proving it against real data

Everything was checked against real account data in a test environment: adding, editing and deleting
a contact on either side behaves correctly with nothing duplicated, and the same checks were repeated
for accounts whose mail lives on a linked secondary server, since accounts don't all keep their mail
in the same place.

Tested (2752/2752).
