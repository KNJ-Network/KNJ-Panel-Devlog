# Phase 202 - The Folder That Was Never Really There

A real, live incident, reported the way most of the good ones are on this project — a screenshot
of a broken inbox, sent in without warning. An account owner had just set up his first
subject-based mail rule, sent a test message to see it work, and from that moment on his webmail
stopped loading his folders at all. His actual mail was fine the whole time; the app just couldn't
see any of it.

## Chasing "could not list folders" to a folder that wasn't real

The app's own logs had the real error waiting, once someone looked: not a connection failure, but
Dovecot flatly refusing a request — "NO Mailbox doesn't exist: dovecot." Something named "dovecot"
was showing up in the account's own folder list, and the moment the app tried to check it like any
other folder, the whole request died.

It turned out to be Dovecot's own bookkeeping, not anything the account owner did wrong. The moment
his rule actually delivered a message for the first time, Dovecot quietly created two files of its
own — a small database and a lock directory — used to avoid double-delivering the same message
twice. Nothing wrong with that on its own. The problem was where they landed: directly inside the
same folder where his real mailboxes live, with dots in their own filenames that Dovecot's folder
scanner reads as a naming convention — "here's a folder called dovecot, and here are two folders
nested inside it." Except no folder called "dovecot" actually exists. It's a phantom, invented
purely from the shape of two filenames that were never meant to be mailboxes at all.

The app had never accounted for a folder like that. It asked every folder the server handed back
for its unread count, one by one, trusting that anything the server listed could actually be
opened. The first time that phantom folder showed up in the list, the whole request failed right
there — not because anything about the account's real mail was broken, but because the one entry
in a list of eight was never meant to be openable in the first place.

## Fixing it so it can't come back a different way

The immediate relief was straightforward: move those two files out of the account's mailbox
folder. Dovecot rebuilds them the moment it needs to again, this time somewhere that doesn't
collide with anything — and folder listing came back instantly, real folders and all, including
the one his own rule had been quietly filing mail into the whole time.

The lasting fix isn't specific to Dovecot's own bookkeeping quirk, on purpose. Every IMAP `LIST`
response can mark an entry `\Noselect` — the protocol's own way of saying "this exists in the
folder hierarchy, but you can't open it." That flag was sitting right there, unread, the entire
time. The fix skips any folder carrying it before ever asking the server for its status, which
means this specific Dovecot behavior can't cause this failure again, and neither can any other
legitimate reason a mail server might hand back a folder that was never meant to be selected.

Tested, full suite green, pint clean. Verified against the real affected account: the inbox loads,
the rule's own folder is right there in the sidebar, and nothing about his mail was ever actually
at risk — the entire incident was the folder *list*, never the mail itself.
