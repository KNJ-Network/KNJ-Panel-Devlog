# Phase 137 - The Half of a Server Switch Nobody Had Built

Linking a Mail Only server and switching the active mail server has always resynced every existing
mailbox's routing and auth data immediately. What it never did, and never claimed to, was move the
mail itself. A mailbox created before the switch stayed where it physically was, and anyone depending
on old mail being reachable afterward had no way to move it short of a manual file sync.

This phase builds the other half: a "Mailbox data migration" card on Mail Settings that runs a real,
queued, live-polled migration. Pick a from and to server (Local or any linked Mail Only server, in
either direction), click Migrate, and watch the log fill in as each mailbox is processed.

The interesting design decision was the transport. The obvious shape was a dedicated upload endpoint
with a raised body limit, but that would have needed a new web-server configuration change that does
not automatically reach servers already running. Instead, each mailbox is archived and split into
small chunks that travel over the existing internal mail-dispatch channel every other mail action uses,
which already fits the default size limit on every server, with no new configuration to keep in sync.

That meant the provisioning runner needed a sibling method that targets a specific server, since a
migration addresses its source and destination independently of whichever server is active.

Two test-suite gotchas turned up along the way, worth noting for next time: the ORM does not read a
migration column's database default back into an in-memory model unless it was passed explicitly, so a
strict zero check compared against null; and PHP arrow functions capture outer variables by value, so a
mock closure referencing an earlier mock's generated ID compared against null forever. Regular
closures fixed it.

Full local suite (2471 tests) and the code-style check green. Deployed to the dev server and confirmed
live: the page renders, the form validates, and the migration-run polling works. A full chunked
transfer between two real linked servers is still to be exercised on a real Mail Only install.
