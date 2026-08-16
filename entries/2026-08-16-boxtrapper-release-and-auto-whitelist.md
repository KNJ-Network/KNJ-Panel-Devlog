# Phase 88 - BoxTrapper Actually Releases Your Mail

Challenge-response has been real since Phase-whatever-that-was: a genuine server-side Sieve rule
diverts anyone not on the allow list into a Pending folder, not an Inbox filter dressed up to look
like one. But approving a sender only ever changed what happened to their *next* message — anything
of theirs already sitting in Pending stayed there until the account owner noticed and moved it by
hand. That's not what BoxTrapper does. Today it does.

## Release-on-approve

`ChallengeResponseService::approve()` already wrote the allow-list row and resynced the mailbox's
Sieve script. It now also calls a new `releasePendingFrom()`, which re-scans Pending (the same
header scan the Pending list view already uses), filters down to messages from the address just
approved, and hands their filenames to a new provisioning action, `mailbox-pending-release`. That
action does the simplest correct thing a Maildir supports: `mv` each message from
`.Pending/{cur,new}/` into the mailbox's own `new/` — landing in `new/` regardless of where it
started, since Dovecot treats anything there as freshly delivered, which is exactly the state a
message the owner has never actually seen should be in. Same filesystem, so no `chown` needed either.

## Auto-whitelist on reply

The second half is smaller but closes a real annoyance: if you've replied to someone, they're
obviously not a stranger, so KNJ Webmail's `send()` now approves the recipient automatically
whenever the outgoing message has an `in_reply_to` and the mailbox has challenge-response turned
on. It reuses the exact same `approve()` call the UI button hits — no separate whitelist path to
keep in sync — and swallows the "already approved" exception rather than treating it as
noteworthy, since replying to someone already on your list should be a no-op, not an error.

## What's still not BoxTrapper

The real cPanel feature also sends the *sender* an automatic challenge email with a link to
confirm themselves, no owner involvement required. That's deliberately not attempted here — it
would mean generating a per-sender confirmation link inside a Sieve `vacation` response, and
Dovecot Pigeonhole's variable-substitution rules for that extension aren't something to guess at.
Approving is still an owner action, from the Pending list or by adding an address directly. A
per-message approve/deny view is also still missing — approval is by sender address, which covers
the common case but not "approve just this one email."

## Next

Continuing down the gap list.
