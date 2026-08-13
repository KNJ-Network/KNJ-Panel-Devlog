# Phase 77 - A Feature That Wasn't There to Build

Next on the gap list was a database root password rotation feature. We checked how the server's
database administrator account actually authenticates before writing code, and the answer changed
the plan.

## Nothing to rotate

The administrator account uses the distribution's default local authentication rather than a
stored password, so there is no credential a UI could rotate. Adding one just to have something to
rotate would introduce a new secret into a system the provisioning flow doesn't need one for, for
a feature that adds no real protection. We decided to mark it not applicable rather than force a
feature to exist where the premise doesn't hold.

## The item that actually was missing

Right next to it on the list was "Manage Database Profiles / Access Hosts", and this one was real.
Account owners could already allow a specific remote IP to connect to their own database users,
self-service, from their Databases page. What was missing was the admin side: a server-wide view
of every database user and the hosts each is allowed to connect from, with the ability to grant or
revoke directly. It reuses the same two service calls the self-service page uses, so there is no
second set of logic to keep in sync. It was verified against a real database user and a real
remote grant: the database account genuinely appeared and genuinely disappeared.
