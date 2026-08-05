# Phase 60 - The Client Domains Section, and Two Bugs Older Than This Build

Three items on the account-side Domains roadmap. All three shipped, and along the way turned up
two real bugs that had nothing to do with the features being built — one in a Sieve fix from the
previous phase, the other in database provisioning that's been there since the Database Browser
first shipped.

Dynamic DNS is the straightforward one: each host gets its own A record and a long, unguessable
update URL — the same token-in-URL pattern the existing leech-check endpoint already uses, since a
router or cron job polling on an interval can't carry an Authorization header the way a browser
can. Verified with a real `dig` query against the nameserver after a real POST to the update URL,
confirming BIND itself answers with the newly reported IP, and confirmed the common case — polling
with an unchanged IP — skips the zone rewrite entirely rather than bumping the serial for nothing.

Quick Site Publisher needed a design decision before any code: the existing "Website Coming Soon"
placeholder is a hardcoded heredoc inside the account-creation and add-domain provisioning actions,
unconditionally rewritten every time either of those actions runs, so anything written there any
other way would eventually get silently clobbered. Built as its own distinct template instead,
written directly into `index.php` — no new provisioning script action needed, since `public_html`
is already group-writable by the panel process for exactly this reason (File Manager and custom
error pages already rely on the same permission). Found live on the very first real publish
attempt that the file also needs no `chmod()`: the placeholder already exists, owned by the
account's own system user, and `chmod()` requires owning the file — group-write access is enough
to rewrite its contents but not to change its permissions. Removed the unnecessary call rather than
working around it.

WordPress management closed out the App Installer's oldest open item: check for updates, apply
them, and clone a working copy to a staging path, all through the same wp-cli invocation pattern
the installer itself already used. Live-verifying it surfaced that wp-cli had never actually been
installed on the dev server at all — the original installer had shipped without ever being proven
against a real WordPress site. Added to `bootstrap-server.sh` the same way Composer already is:
official phar, checksum verified against wp-cli's own published sha512 before it's trusted. That
unblocked a second, older bug: MySQL and MariaDB process backslash escape sequences inside a
single-quoted string literal by default, so a generated database password containing a literal
backslash was silently turned into a *different* effective stored password by `CREATE USER` than
the one written into `wp-config.php` — breaking the connection with no error anywhere at creation
time. Reproduced it directly (a raw `CREATE USER ... IDENTIFIED BY 'abc\def'` really does store
something other than `abc\def`), fixed by escaping the backslash and the quote before the password
ever reaches the SQL literal, and reproduced-then-confirmed the fix with the same round-trip. This
bug predates WordPress management entirely — it's been sitting in `db-user-create` and
`db-user-password` since the Database Browser first shipped, just never hit because no earlier
generated password happened to contain a backslash.

Three real features, two real bugs found the same way as always — by insisting on live verification
against the real server rather than trusting a green test suite alone. Domains section closed out.
