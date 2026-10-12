# Phase 255 - Two Lists, One Click

Sign-ups for the panel's own update list had started arriving on their own. That is the moment a manual
process stops being fine.

## The export-import routine

The panel has two separate things that both look like "a list of email addresses". A subscriber list is the
marketing side: sign-up forms, double opt-in, campaigns, unsubscribes. A mailing list is a mail address on a
site that forwards whatever is sent to it to every member. To send an update to all subscribers, the only
route was to export one and import it into the other, every time.

## Linking them

A subscriber list now has a Linked mailing list card. One form creates the mailing list and links it, fills
it with the current subscribers, and from then on keeps it in step: new sign-ups, confirmations, imports and
unsubscribes all follow. Sending an update becomes an email to that address.

The part worth writing down is that none of this happens while the visitor waits. Adding a member to a
mailing list can take many seconds, so every change is queued and handled in the background, plus a full
reconcile on link and on a Sync now button. Deleting the mailing list unlinks it automatically; unlinking
never deletes it.
