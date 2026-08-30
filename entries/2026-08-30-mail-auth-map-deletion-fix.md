# Phase 164 - The Account That Wouldn't Leave

During readiness testing, we created a disposable account with one mailbox, confirmed DNS sync and mail delivery, then deleted it and checked that nothing was left behind. DNS was clean everywhere. On the linked Mail Only server, however, the deleted account's mail routing entries were still present.

## Why

Account deletion removes mailboxes through a database cascade, which does not trigger the normal mail-server sync that every other mailbox change performs. DNS has a periodic reconciliation that would have caught this; mail did not.

## The fix

Account deletion now refreshes the mail routing data for mailboxes, catch-alls, mailing lists and forwarders. Each refresh is best effort and logged, so an unreachable mail server can never leave an account stuck in deleting. Two regression tests cover this. All four paths were fixed together, since the same pattern applied to each.

Tested (2563/2563).
