# Phase 07 - A Real Database Manager, Then an App Installer Built On Top Of It

M4 was originally scoped as just an app installer — a one-click way to get WordPress onto a
provisioned site. Partway into planning it, a fair question came up: the installer needs a
database to work at all, so shouldn't the panel have a real database manager first, rather than
have the installer quietly create one behind the scenes with nothing for the account owner to
actually see or manage?

That reordered the milestone. Database manager first, app installer built directly on top of it.

## The Database Manager

Split into three ideas that work together: databases, MySQL users, and the grants that connect a
user to a database with a specific set of privileges. A user isn't locked to one database, and a
database can have more than one user — a read-only reporting login alongside the main application
login, for instance. The account side of the panel now has exactly that: create and delete
databases, create and delete database users with independent passwords, and grant either full
access or a specific hand-picked set of privileges to a particular database/user pair.

Getting there meant restructuring how the panel stored this internally, since the original design
allowed exactly one database and one user per account. Existing databases were carried over into
the new shape rather than lost.

Every new operation got the same treatment as everything else that touches the server: tested on
purpose with hostile input, including injection attempts in names and privilege strings and
attempts to reach another account's database. All were correctly rejected before anything
privileged ran.

Two real bugs came out of that testing: a validation rule on a checkbox that could never fire in
real browsers, and a revoke operation that reported success without actually revoking. Both fixed
and re-verified.

## The App Installer

With that in place, installing WordPress needed no special-case database logic — it calls the same
database manager an account owner would use. Pick a subdirectory, a site title, and an admin
login, and the panel creates the database, installs WordPress for real, and hands back a working
site.

Verified end to end against a live development server: a genuine install, a real login to the
installed WordPress admin, and removal that deletes the files, the database, and the user. A
deliberately failed install rolls back everything it had started creating, confirmed by checking
for leftovers. There weren't any.
