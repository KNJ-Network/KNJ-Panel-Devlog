# Phase 88 - BoxTrapper Actually Releases Your Mail

Challenge-response has been working for a while: a server-side mail rule diverts anyone not on the
allow list into a Pending folder. But approving a sender only changed what happened to their next
message, and anything already in Pending stayed there until the account owner moved it by hand.
That's not what BoxTrapper does. Now it does.

## Release-on-approve

Approving a sender already wrote the allow-list entry and resynced the mailbox's filter. It now
also finds messages from the approved address in Pending and delivers them to the inbox as new
mail, which is the right state for a message the owner has never actually seen.

## Auto-whitelist on reply

The second half closes a real annoyance: if you've replied to someone, they're clearly not a
stranger. Webmail's send path now approves the recipient automatically when the outgoing message
is a reply and challenge-response is on. It reuses the same approval call the UI button uses, and
replying to someone already approved is a no-op rather than an error.

## What's still not BoxTrapper

The real cPanel feature also sends the sender an automatic challenge email with a confirmation
link, with no owner involvement. That isn't attempted here, so approval remains an owner action,
from the Pending list or by adding an address directly. A per-message approve/deny view is also
missing, since approval is by sender address.
