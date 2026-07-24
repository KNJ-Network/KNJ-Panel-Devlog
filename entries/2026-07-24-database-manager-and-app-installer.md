# A Real Database Manager, Then an App Installer Built On Top Of It — 24 July 2026

M4 was originally scoped as just an app installer — a one-click way to get WordPress onto a
provisioned site, the same job Softaculous does on real cPanel. Partway into planning it, a fair
question came up: the installer needs a database to work at all, so shouldn't the panel have a
real database manager first, rather than have the installer quietly create one behind the
scenes with nothing for the account owner to actually see or manage?

That reordered the milestone. Database manager first, app installer built directly on top of it.

## The Database Manager

cPanel splits this into three ideas that work together: databases, MySQL users, and the grants
that connect a user to a database with a specific set of privileges. A user isn't locked to one
database, and a database can have more than one user — a read-only reporting login alongside the
main application login, for instance. The account side of the panel now has exactly that: create
and delete databases, create and delete database users with independent passwords, and grant
either full access or a specific hand-picked set of privileges (the same list cPanel exposes —
SELECT, INSERT, UPDATE, and so on) to a particular database/user pair.

Getting there meant restructuring how the panel stored this internally — the original design
baked exactly one database, one user, and one password into a single row, which doesn't leave
room for any of the above. That got split into three connected tables, with the database that
already existed for every account (created automatically the moment an account is provisioned)
carried over into the new shape rather than lost.

Every new privileged operation this needed — create/delete a database, create/delete a user,
change a password, grant, revoke — got the same treatment as everything else that talks to the
server: attacked on purpose before being called done. SQL and shell injection attempts in
database names and privilege strings, attempts to reach across and modify another account's
database by guessing its ID — all correctly rejected, with the privileged operation never even
attempted once the input failed validation.

Two real bugs came out of that testing, not just theoretical ones. An "unchecked checkbox"
validation rule that could never actually have fired in real use, because browsers don't send
what the code was checking for. And a database revoke command using MariaDB syntax that's
actually invalid — it had been failing every single time, silently, because the error was being
thrown away rather than surfacing. Revoking access reported success while quietly doing nothing
at all. Both fixed, both re-verified live.

## The App Installer

With that in place, installing WordPress stopped needing any special-case database logic of its
own — it just calls the same database manager an account owner would use themselves. Pick a
subdirectory, a site title, and an admin login, and the panel creates the database, installs
WordPress into it for real via `wp-cli`, and hands back a working site.

Verified end to end against the live server: a genuine `wp-cli` install, a real login into the
installed WordPress admin, the site reachable over the public internet on its real domain, and —
just as important — removal that actually deletes the files, the database, and the user, rather
than leaving anything behind. A failed install (tested by deliberately pointing it at an
already-used location) rolls back everything it had started creating, confirmed by checking the
real database server afterward for orphaned leftovers. There weren't any.

## Next

M5 — Mail Provisioning, and eventually M6 for DNS. Both are big enough to warrant their own pass
rather than folding in early.
