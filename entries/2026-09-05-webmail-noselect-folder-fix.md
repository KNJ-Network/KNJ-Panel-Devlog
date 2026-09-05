# Phase 202 - The Folder That Was Never Really There

A broken webmail inbox turned up during live use: after a first subject-based mail rule delivered its
first message, the account's folders stopped loading. The mail itself was fine; the app just couldn't
see any of it.

## Chasing "could not list folders" to a folder that wasn't real

The app's logs held the real error: the mail server refused a request for a mailbox that doesn't exist.
An entry named after the mail server's own bookkeeping was appearing in the folder list, and the moment
the app tried to check it like any other folder, the whole request failed.

The first time a rule delivered a message, the mail server created two bookkeeping items for
duplicate-delivery tracking. They landed alongside the real mailboxes, and their names are read by the
folder scanner as a naming convention, producing a phantom folder that was never meant to be opened.

The app had never accounted for this. It asked every folder the server listed for its unread count,
trusting that anything listed could be opened, so one unopenable entry failed the whole list.

## Fixing it so it can't come back a different way

The immediate relief was to move the stray items out of the way, after which folder listing came back
immediately with the real folders, including the one the rule had been filing mail into.

The lasting fix isn't specific to that bookkeeping quirk. Every IMAP `LIST` response can mark an entry
`\Noselect`, the protocol's own way of saying "this exists in the hierarchy but can't be opened". That
flag had been ignored. The app now skips such entries before requesting their status, which covers this
case and any other legitimate reason a server might list an unselectable folder.

Tested, full suite green, pint clean, and verified against an affected account: the inbox loads and the
rule's folder is in the sidebar. The mail itself was never affected; the problem was the folder list.
