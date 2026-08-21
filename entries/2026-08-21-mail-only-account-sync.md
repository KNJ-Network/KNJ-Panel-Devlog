# Phase 106 - Mail Only: Syncing Existing Accounts to a New Mail Server

## The gap behind "existing mailboxes aren't copied over yet"

Stage 2's Mail Settings selector shipped with an honest caveat in its UI: switching to a linked
Mail Only server did nothing for mailboxes that already existed. The cause was bigger than a
missing resync: on Main, the mail stack looks up mailboxes and password hashes in the panel's own
database, and a separate Mail Only box has no local copy of that database. Even a successful
mailbox-create dispatched to it would have left no way to authenticate.

## Same pattern as DNS, applied one level deeper

The fix follows the same shape as the rest of this work: never share a live database connection
across the link; push generated configuration instead. A mail auth-map service builds flat files
from Main's current mailbox and domain state and pushes them over the dispatcher built in Stage 2.

It always rebuilds the full current state rather than applying increments, the same convention
DNS zone-membership pushes use. That closes the original gap: regenerating once a server is
selected replays every existing mailbox, with no separate bulk-sync job. It is also wired into
mailbox create, delete and password change so a linked server stays current. On an install that
has never selected a Mail Only server it does nothing, keeping behavior unchanged until an admin
opts in.

## The mail-server side, new ground

The mail setup gained a Mail Only-specific branch that points the mail servers at the pushed files
instead of the SQL configuration Main uses. This is new systems configuration, not covered by the
PHP test suite. It follows the same backup, validate and revert-on-failure discipline as other
configuration-writing actions, and was syntax-checked; live verification awaited a real linked
Mail Only box.

## One more thing found along the way

Server Setup's deferred certificate-issuance flow was running both the mail setup and the DNS
setup regardless of role, a pre-existing gap. It now reads the install's role and runs only what
applies to it.

## Explicitly not done in this pass

Forwarders, catch-alls and mailing lists had the same live-database problem and were not covered
here; the account-sync UI copy and this entry say so plainly.

## Verified

Full local suite (1597 tests) and the code-style check green, with new tests for the auth-map
service. No live VM test yet at this stage. Committed and pushed.
