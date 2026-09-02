# Phase 179 - The Address Book Nobody Went Looking For

A real customer's own son restored his account from a real backup, checked his mail, checked his
files, checked his subdomains — everything was there. Then he noticed his saved contacts weren't.
Nobody had built anything to look for them, because nobody had checked whether a full-account backup
even carried them. It does.

## A whole database, sitting quietly next to the mail

cPanel/WHM's own full-account backup stores webmail's saved contacts the same way it stores mail
itself: on disk, per mailbox, no export step required. What's actually there is a complete, standalone
SQLite database — `<mailbox>.rcube.db.latest`, a symlink to a timestamped snapshot — carrying
Roundcube's own real tables: contacts, contact groups, group membership, even the vCard and search-
index text for each one. Confirmed against a real archive before writing a line of code, the same way
every other piece of this restore feature has been: not a placeholder, not empty, a real address book
with real groups.

## Two webmail clients, one clear scope

This panel supports two different webmail clients per account, and they don't share a contacts
format — one has a rich, groups-and-vCard address book; the other, the panel's own simpler
alternative, stores just a name and an email with no concept of groups at all. Importing needed a
clear line, not an attempt to serve both from one code path: every cPanel-restored account starts on
the richer client by default, and that's also the one the archive's own data already matches natively.
Built for that one, skipped cleanly (not as an error) for the other — a real need for the second path
would be its own, differently-shaped piece of work, not a shortcut bolted onto this one.

## Getting the data across without losing anyone's identity

The two ends of this transfer don't share numeric ids — a contact that was id 3 in the old, standalone
file has no reason to land as id 3 in a live, shared database with contacts from other accounts
already in it. Each insert is written to look itself up immediately afterward by something that
actually identifies it — an email address for a contact, a name for a group — so every later
statement that needs to reference "the group I just created" or "the contact I just created" gets the
real, current id back, not a guess. The same lookup-instead-of-assume shape also makes a second run of
the same import a genuine no-op: nothing is inserted twice, because everything checks for its own
existence first.

## Verifying it against the account that started this

Every unit test uses a real SQLite database built with the exact table shape a real archive carries,
not a hand-waved approximation — including one contact with a name containing an apostrophe, to prove
the generated SQL survives content that would break a naively-built query. Then, on panel-dev, against
the same real archive whose owner reported the original gap: all 6 real contacts, all 3 real groups,
and all 6 real group memberships landed exactly as they were on the old host — including correctly
creating a fresh webmail login row for a mailbox that had never actually opened webmail there before.
Ran the same import again immediately after. Nothing duplicated.

Tested (2699/2699).
