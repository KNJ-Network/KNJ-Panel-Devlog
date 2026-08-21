# Phase 105 - Mail Only, Stage 2: The Dispatcher, and a Real Selector

## The piece DNS-only never needed

DNS-only's design rests on zone content always being authored on Main and pushed. Mail does not
get that luxury. Creating a mailbox or changing a password is something an admin triggers inside a
single request and expects a real result from immediately. Genuine mail provisioning meant
building a way to redirect a privileged action to a different box and get its result back
synchronously.

## One interception point, not twenty

Roughly twenty service classes call the same privileged provisioning step. Rather than teach each
about a linked Mail Only server, a single provisioning runner makes the decision: local, unless a
Mail Only server is actively selected, in which case the action is dispatched instead. Every
existing caller is unchanged, and installs that have not opted in see no behavior difference,
confirmed by existing tests passing without edits.

The remote side reuses DNS-only's established shape: a synchronous authenticated request to an
internal endpoint on the Mail Only box, with the same hostname-then-IP fallback the DNS push uses.
The receiving side needs no mail-specific logic; it runs the same runner locally.

Handling of temporary files for secret values, previously duplicated at several call sites, now
lives once inside the runner, and turned out to suit non-secret content such as generated mail
server configuration too.

## Proof case: mailbox create/delete, and one real feature besides

Mailbox handling is the proof case, covered by tests for both local and remote branches, including
a check that a stale server selection fails safe to local. This pass also shipped the user-facing
switch: a "Where does mail run?" selector on Mail Settings, offering Local or any linked Mail Only
server that has completed the handshake. Switching re-applies the current settings immediately. The
page says plainly that existing mailboxes are not copied over automatically.

## Verified

Full local suite (1593 tests) and the code-style check green. No live VM test at this stage;
the code is covered by unit and feature tests with faked processes and HTTP. Committed and pushed.
